# Análise de Detecção — Windows Authentication Brute Force

## 1. Resumo do Incidente

Foi realizada uma simulação controlada de múltiplas tentativas de autenticação contra um endpoint Windows dentro do ambiente de laboratório.

O objetivo foi validar a capacidade do Wazuh de:

- Coletar eventos de falha de autenticação;
- Identificar a origem das tentativas;
- Correlacionar múltiplos eventos;
- Gerar um alerta de segurança;
- Fornecer informações suficientes para investigação.

---

## 2. Origem da Atividade

O tráfego utilizado durante a simulação teve origem no host Kali Linux:

```text
Source IP: 192.168.56.103
```
O endpoint Windows monitorado estava configurado com:
```text
Windows IP: 192.168.56.104
```
A comunicação ocorreu dentro da rede privada utilizada pelo laboratório.
---

## 3. Reconhecimento

Antes da simulação das autenticações, foi realizado reconhecimento do endpoint utilizando Nmap.

O objetivo foi identificar serviços de rede disponíveis e determinar quais serviços poderiam ser utilizados durante o teste.

A enumeração identificou o serviço SMB no endpoint Windows.

A evidência do reconhecimento:
<img width="828" height="344" alt="001-nmap-smb-enumeration" src="https://github.com/user-attachments/assets/5db62a1b-cd9f-436e-a3fc-a8fdd775e93e" />

---

## 4. Simulação das Tentativas de Autenticação

Após o reconhecimento, foram realizadas múltiplas tentativas de autenticação contra o endpoint Windows utilizando o Kali Linux.

As tentativas foram realizadas de forma controlada com credenciais inválidas, com o objetivo de gerar eventos de falha de autenticação.

A evidência da atividade:

<img width="1512" height="383" alt="002-Kali-attack" src="https://github.com/user-attachments/assets/70d5e7a0-abdb-4050-a0d3-dcc126f333f8" />

---

## 5. Evento de Segurança Windows

As tentativas de autenticação geraram eventos de segurança no Windows.

O principal evento analisado foi:

```text
Event ID: 4625
Description: An account failed to log on
```
O evento apresentou os seguintes indicadores:


| Campo                     | Valor          |
| ------------------------- | -------------- |
| Event ID                  | 4625           |
| TargetUserName            | joao           |
| LogonType                 | 3              |
| AuthenticationPackageName | NTLM           |
| IpAddress                 | 192.168.56.103 |
| IpPort                    | 53586          |
| Status                    | 0xc000006d     |
| SubStatus                 | 0xc000006a     |

Logon Type

O valor:
```text
LogonType: 3
```
representa um Network Logon, indicando que a tentativa de autenticação ocorreu através da rede.

Source IP

O campo:
```text
IpAddress: 192.168.56.103
```
permitiu identificar o endereço de origem da tentativa.

Esse endereço corresponde ao host Kali utilizado para a simulação.

A evidência do evento Windows:

<img width="666" height="656" alt="003-Windows-Event-4625" src="https://github.com/user-attachments/assets/9d85af56-7c93-4c17-b76b-f4e8bd4dc6c8" />

---

## 6. Coleta pelo Wazuh

O Wazuh Agent instalado no Windows coletou os eventos de segurança e os encaminhou para o Wazuh Manager.

O evento 4625 foi inicialmente identificado através da regra:
```text
Rule ID: 60122
Description: Logon Failure - Unknown user or bad password
```
Essa regra representa a identificação de uma falha individual de autenticação.

A evidência da detecção inicial:

<img width="1855" height="666" alt="004-wazuh-bruteforce-detection" src="https://github.com/user-attachments/assets/f86eda74-4244-4748-80c5-f5df5e56a118" />

---

## 7. Correlação dos Eventos

Eventos individuais de falha de autenticação podem representar situações legítimas, como um usuário digitando uma senha incorreta.

Por esse motivo, o laboratório utiliza uma lógica de correlação para identificar um padrão de múltiplas falhas provenientes da mesma origem.

Threshold utilizado
```text
5 tentativas de autenticação malsucedidas
+
mesmo endereço IP
+
janela de 60 segundos definida
```
Quando o padrão é identificado, o Wazuh gera um alerta específico para o comportamento de força bruta.

---

## 8. Regra Personalizada

Foi criada uma regra personalizada no Wazuh:
```text
<group name="windows,authentication,bruteforce,">

  <rule id="100100" level="12" frequency="5" timeframe="60">
    <if_matched_sid>60122</if_matched_sid>
    <same_field>win.eventdata.ipAddress</same_field>
    <description>Windows Authentication Brute Force - Multiple failed logons from same source IP</description>
    <mitre>
      <id>T1110.001</id>
    </mitre>
  </rule>

</group>
```
A regra permite diferenciar uma falha isolada de autenticação de um comportamento repetitivo compatível com brute force.

A evidência da regra personalizada:

<img width="1589" height="288" alt="005-Wazuh-rule-100100" src="https://github.com/user-attachments/assets/505c7072-36f7-41bc-97ad-6bb16986cde1" />

---

## 9. Alerta Gerado

Após a geração das múltiplas falhas de autenticação, a regra personalizada foi acionada.

## Alerta

```text
Rule ID: 100100
Description: Windows Authentication Brute Force
```
Indicadores observados:
```text
Source IP:       192.168.56.103
Target User:     joao
Logon Type:      3
Event ID:        4625
Authentication:  NTLM
```
Esses indicadores permitem ao analista iniciar a investigação da atividade sem depender exclusivamente da mensagem descritiva do alerta.

A evidência do alerta:

<img width="1843" height="227" alt="006-Wazuh-alert-details" src="https://github.com/user-attachments/assets/24fdc844-7163-4436-b183-f2299c7c5fad" />

---

## 10. Timeline da Atividade

O fluxo observado durante o laboratório foi:

```text
[1] Reconhecimento
        │
        ▼
     Nmap
        │
        ▼
[2] Serviço identificado
        │
        ▼
[3] Tentativas de autenticação
        │
        ▼
[4] Windows Event ID 4625
        │
        ▼
[5] Wazuh Agent
        │
        ▼
[6] Wazuh Manager
        │
        ▼
[7] Regra 60122
        │
        ▼
[8] Correlação de eventos
        │
        ▼
[9] Regra 100100
        │
        ▼
[10] Alerta de Brute Force
```
---

## 11. Análise do Analista

Com base nos eventos observados, foram identificados os seguintes elementos:

Origem:
```text 
192.168.56.103
```
Host Kali utilizado na simulação.

Alvo 
```text
192.168.56.104
```
Endpoint Windows monitorado.

Usuário: 
```text 
joao
```
Conta utilizada como alvo das tentativas de autenticação.

Evento: 
```text 
4625
```
Falha de autenticação registrada pelo Windows.

Comportamento
Múltiplas falhas de autenticação provenientes do mesmo endereço IP dentro da janela de correlação configurada.


### Veredito do Analista

**Classificação:** Verdadeiro Positivo

**Severidade:** Média

**Confiança:** Alta

**Avaliação:**

A atividade observada é consistente com uma tentativa de
força bruta de senha originada do host Kali Linux.

A avaliação é fundamentada por:

- Múltiplas tentativas de autenticação malsucedidas;
- Mesmo IP de origem em todos os eventos;
- Tipo de Logon de Rede 3;
- ID de Evento do Windows 4625;
- Mesma conta de destino;
- Correlação dentro da janela de tempo configurada.

**Conclusão:**

A atividade é classificada como um Verdadeiro Positivo para
ataque de força bruta de autenticação no Windows no
ambiente de laboratório controlado.

---

## 12. Classificação

O comportamento observado é compatível com uma tentativa de:

```text 
Brute Force / Password Guessing
```
A atividade foi realizada em ambiente controlado e não representa um ataque contra sistemas externos.

## 13. MITRE ATT&CK

A atividade está relacionada à técnica E Sub-técnica:

**Técnica** - T1110 — Brute Force - [Link Mitre ATT&CK](https://attack.mitre.org/techniques/T1110/) | **sub-técnica (T1110/001)** - [Link Mitre ATT&CK](https://attack.mitre.org/techniques/T1110/001)

A técnica T1110 descreve tentativas de obter acesso através de tentativas repetidas de autenticação.

O cenário deste laboratório representa especificamente um comportamento da sub-técnica **T1110/001** de Password Guessing / Brute Force.

---

## 14. Resposta e Contenção

Durante esta etapa do projeto, a prioridade foi validar a cadeia de:
```text
Telemetria
    ↓
Detecção
    ↓
Correlação
    ↓
Alerta
    ↓
Investigação
```
A implementação do Wazuh Active Response para bloqueio automático do endereço IP será realizada e validada em uma etapa posterior.

A pasta relacionada à resposta está localizada em:
```text
response/
```

## 15. Conclusão

O laboratório demonstrou com sucesso a capacidade de transformar eventos individuais de falha de autenticação em um alerta contextualizado de segurança.

O processo validado foi:
```text
Kali Linux
     ↓
Tentativas de autenticação
     ↓
Windows Event ID 4625
     ↓
Wazuh Agent
     ↓
Wazuh Manager
     ↓
Regra 60122
     ↓
Correlação
     ↓
Regra 100100
     ↓
Windows Authentication Brute Force
     ↓
Investigação
```
O cenário demonstra conceitos fundamentais de uma operação de SOC / Blue Team, incluindo coleta de telemetria, análise de logs, identificação de indicadores, correlação de eventos, detecção e investigação de incidentes.

---

| Evidência                  | Arquivo                                         |
| -------------------------- | ----------------------------------------------- |
| Reconhecimento Nmap        | `screenshots/01-nmap-smb-enumeration.png`       |
| Simulação no Kali          | `screenshots/02-kali-smb-bruteforce.png`        |
| Windows Event 4625         | `screenshots/03-windows-event-4625.png`         |
| Detecção no Wazuh          | `screenshots/04-wazuh-bruteforce-detection.png` |
| Regra personalizada 100100 | `screenshots/05-wazuh-rule-100100.png`          |
| Alerta final               | `screenshots/06-wazuh-alert-details.png`        |



