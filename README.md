<img width="1823" height="863" alt="RovaniSec-Lab" src="https://github.com/user-attachments/assets/4a13b04f-a376-4002-87eb-ac363b00a49f" />

# SOC Lab — Brute Force Detection & Incident Investigation

Laboratório prático de **Blue Team / SOC** desenvolvido para simular, detectar e investigar tentativas de autenticação por força bruta contra um endpoint Windows utilizando **Wazuh SIEM**.

O projeto reproduz um cenário de ataque controlado, desde o reconhecimento do serviço exposto até a geração e investigação do alerta de segurança.

---

## 🎯 Objetivo

Construir um ambiente de laboratório capaz de:

- Simular tentativas de autenticação mal-sucedidas;
- Gerar telemetria de segurança no Windows;
- Identificar eventos de falha de autenticação;
- Coletar os eventos através do Wazuh Agent;
- Correlacionar múltiplas tentativas provenientes do mesmo endereço IP;
- Gerar um alerta personalizado no Wazuh;
- Investigar a origem, usuário alvo e tipo de logon;
- Documentar o processo de investigação;
- Mapear a atividade para o MITRE ATT&CK;
- Avaliar mecanismos de contenção e resposta automatizada.

---



## Tecnologias utilizadas

Tecnologia	Função
Wazuh	SIEM, correlação e detecção
Wazuh Agent	Coleta de eventos do Windows
Wazuh Dashboard	Investigação e visualização dos alertas
Windows 10	Endpoint monitorado
Windows Event Logs	Fonte principal de telemetria de autenticação
Sysmon	Telemetria adicional do endpoint
Kali Linux	Simulação do atacante
Nmap	Reconhecimento e identificação de serviços
NetExec	Simulação de tentativas de autenticação
PowerShell	Administração e validação do ambiente
MITRE ATT&CK	Classificação da atividade
🔴 Cenário do Ataque

O laboratório simula um atacante realizando múltiplas tentativas de autenticação contra um serviço exposto no endpoint Windows.

O fluxo utilizado foi:

```text

Kali Linux
     │
     │ Reconhecimento
     ▼
   Nmap
     │
     │ Serviço SMB identificado
     ▼
Windows 10
     │
     │ Tentativas de autenticação
     ▼
Event ID 4625
     │
     │ Wazuh Agent
     ▼
Wazuh Manager
     │
     │ Correlação
     ▼
Regra personalizada 100100
     │
     ▼
🚨 Windows Authentication Brute Force

```
---

1️⃣ Reconhecimento do serviço

O primeiro passo foi realizar o reconhecimento do endpoint utilizando o Nmap.

O objetivo foi identificar serviços de rede disponíveis antes da simulação das tentativas de autenticação.

Evidência:

<img width="828" height="344" alt="001-nmap-smb-enumeration" src="https://github.com/user-attachments/assets/69887534-9ca3-44c1-8990-282d168db9ea" />


A enumeração permitiu identificar o serviço SMB exposto no endpoint Windows.

2️⃣ Simulação das tentativas de autenticação

Após identificar o serviço, foram realizadas múltiplas tentativas de autenticação utilizando o Kali Linux.

O objetivo não foi obter acesso ao sistema, mas gerar eventos de falha de autenticação para alimentar o processo de detecção.

Evidência

<img width="1512" height="383" alt="002-Kali-attack" src="https://github.com/user-attachments/assets/16d71ee7-8bdd-4cf4-89d0-850f2f782be1" />


As tentativas foram realizadas contra o usuário utilizado no laboratório, gerando falhas de autenticação no Windows.

3️⃣ Análise da telemetria Windows

As tentativas de autenticação foram registradas pelo Windows Security Event Log através do Event ID 4625.

O evento analisado apresentou os seguintes dados:

Campo	Valor
Event ID	4625
Target User	joao
Logon Type	3
Authentication	NTLM
Source IP	192.168.56.103
Source Port	53586
Status	0xc000006d
SubStatus	0xc000006a

O Logon Type 3 indica uma tentativa de autenticação realizada através da rede.

O endereço 192.168.56.103 corresponde ao host Kali utilizado no laboratório.

Evidência

<img width="666" height="656" alt="003-Windows-Event-4625" src="https://github.com/user-attachments/assets/2fb0a53f-ebcd-4a25-8e55-5ed5bf5bb13c" />


4️⃣ Detecção no Wazuh

Os eventos de segurança foram coletados pelo Wazuh Agent e enviados ao Wazuh Manager.

O Wazuh identificou inicialmente os eventos individuais de falha de autenticação através da regra:

Rule ID: 60122
Description: Logon Failure - Unknown user or bad password

A partir desses eventos foi construída uma lógica de correlação para identificar múltiplas falhas provenientes da mesma origem.

Evidência:

<img width="1855" height="666" alt="004-wazuh-bruteforce-detection" src="https://github.com/user-attachments/assets/520f7fdb-cf6a-43d2-9681-396fc14e2414" />


5️⃣ Regra personalizada de detecção

Foi criada uma regra personalizada no Wazuh com o ID:

100100

A regra foi desenvolvida para identificar o comportamento de múltiplas falhas de autenticação provenientes da mesma origem dentro de uma determinada janela de tempo.

## Condição de detecção

5 falhas de autenticação
        +
mesmo endereço IP
        +
janela de tempo definida
        ↓
Windows Authentication Brute Force

Evidência:

<img width="1589" height="288" alt="005-Wazuh-rule-100100" src="https://github.com/user-attachments/assets/087cb3b0-b306-4ddb-a585-9e2377403591" />


A criação da regra demonstra a utilização de correlação de eventos para transformar eventos individuais em um alerta de segurança contextualizado.

6️⃣ Alerta de Brute Force

Após a geração das múltiplas falhas de autenticação, o Wazuh acionou a regra personalizada:

Rule ID: 100100
Description: Windows Authentication Brute Force

O alerta permitiu identificar os principais indicadores da atividade:

Source IP:       192.168.56.103
Target User:     joao
Logon Type:      3
Event ID:        4625
Authentication:  NTLM

Evidência:

<img width="1843" height="227" alt="006-Wazuh-alert-details" src="https://github.com/user-attachments/assets/c5996e24-67a5-4898-90a5-1e409a794e15" />

---
## Processo de Investigação

A investigação segue um fluxo semelhante ao utilizado em operações de SOC:

                 🚨 ALERTA
                    │
                    ▼
          Identificar o evento
                    │
                    ▼
          Identificar origem
                    │
                    ▼
          Identificar usuário
                    │
                    ▼
        Analisar tipo de logon
                    │
                    ▼
       Correlacionar os eventos
                    │
                    ▼
       Determinar comportamento
                    │
                    ▼
       Classificar o incidente
                    │
                    ▼
          Contenção / resposta
                    │
                    ▼
             Documentação
---

## Indicadores identificados

Origem: 192.168.56.103

Usuário alvo: joao

Evento: 4625 - Failed Logon

Tipo de logon: 3 - Network Logon

Autenticação: NTLM

## Análise do Incidente

A sequência de eventos apresenta características compatíveis com uma tentativa de brute force / password guessing:

Um host remoto realizou múltiplas tentativas de autenticação;
As autenticações foram rejeitadas;
O Windows registrou os eventos 4625;
Os eventos foram enviados ao Wazuh;
O Wazuh identificou a recorrência das falhas;
A regra personalizada 100100 foi acionada;
O alerta permitiu identificar a origem e o usuário alvo.

O laboratório demonstra, portanto, o processo de transformação de telemetria bruta em um alerta contextualizado de segurança.

---

## Resposta e Mitigação

Como parte da estratégia de resposta, o laboratório considera o bloqueio temporário do endereço IP identificado como origem da atividade.

A implementação de mecanismos de resposta automatizada deve considerar:

Falsos positivos;
Endereços IP confiáveis;
Threshold de tentativas;
Duração do bloqueio;
Possibilidade de reversão;
Auditoria das ações executadas.

A resposta automatizada deve ser aplicada com cautela para evitar o bloqueio indevido de usuários legítimos ou sistemas confiáveis.

##  ATT&CK

A atividade simulada está relacionada a técnicas de tentativa de obtenção de credenciais através de autenticações repetidas.

## T1110/001 — Brute Force [Link  ATT&CK](https://attack..org/techniques/T1110/001)

A técnica T1110/001 - Brute Force descreve tentativas sistemáticas de autenticação utilizando diferentes credenciais ou combinações de credenciais.

Dentro do laboratório, o comportamento observado é compatível com password guessing / brute force de autenticação.

## Estrutura do Projeto

```text
soc-lab-bruteforce-detection/
│
├── README.md
│
├── screenshots/
│   ├── 01-nmap-smb-enumeration.png
│   ├── 02-kali-smb-bruteforce.png
│   ├── 03-windows-event-4625.png
│   ├── 04-wazuh-bruteforce-detection.png
│   ├── 05-wazuh-rule-100100.png
│   └── 06-wazuh-alert-details.png
│
├── lab/
│   └── architeture.md
│
├── response/
│   └── active-response.md
│
└── docs/
    └── detection-analysis.md
```
---

## Competências Demonstradas

### Este laboratório demonstra conhecimentos práticos em:

Monitoramento de endpoints Windows;
Windows Security Event Logs;
Event ID 4625;
Análise de falhas de autenticação;
Identificação de origem de eventos;
Wazuh SIEM;
Wazuh Agent;
Criação de regras personalizadas;
Correlação de eventos;
Investigação de alertas;
Análise de indicadores;
MITRE ATT&CK;
Conceitos de Blue Team;
Fundamentos de SOC;
Processo de triagem e investigação de incidentes.

## Próximos passos

Possíveis evoluções do laboratório:

Implementar e validar o Active Response do Wazuh;
Adicionar bloqueio temporário do IP atacante;
Criar uma whitelist para endereços confiáveis;
Melhorar a correlação entre diferentes tipos de autenticação;
Adicionar dashboards específicos para autenticação;
Criar uma timeline automatizada do incidente;
Expandir a detecção para RDP e SSH;
Documentar procedimentos de resposta a incidentes.

Autor: João Rovani

Estudante de Segurança da Informação | Blue Team | SOC | Wazuh | Sysmon | Análise de Logs
