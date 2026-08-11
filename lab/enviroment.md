# Lab Environment

## Objetivo

Documentar a infraestrutura utilizada no laboratório de
detecção e resposta a ataques de força bruta.

---

## Arquitetura

```text
┌─────────────────────┐
│     Kali Linux      │
│                     │
│      Attacker       │
│                     │
│ Hydra / Nmap        │
└──────────┬──────────┘
           │
           │ Attack
           ▼
┌─────────────────────┐
│    Target System    │
│                     │
│   Windows / Linux   │
│                     │
│   Wazuh Agent       │
│   Sysmon            │
└──────────┬──────────┘
           │
           │ Security Logs
           ▼
┌─────────────────────┐
│    Wazuh Manager    │
│                     │
│        SIEM         │
│                     │
│ Detection &         │
│ Active Response     │
└─────────────────────┘
```

## Componentes

### Attacker

*** Sistema: *** Kali Linux
*** Função: *** Simulação do atacante

*** Ferramentas: ***
 - Hydra
 - Nmap
 - SSH Client


### Target

*** Sistema: *** [preencher]
*** IP: ***  [preencher]
*** Função: *** Sistema alvo do ataque.

*** Componentes: ***
 - Wazuh Agent
 - Sysmon
 - Serviço monitorado

---

 ### Wazuh Manager

 *** Sistema: *** [preencher]
 *** Ip: *** [preencher]

 *** Função: ***
  1. Coleta de eventos
  2. Análise de logs
  3. Detecção
  4. Correlação
  5. Alertas
  6. Active Response

---

  ## Network 

```
   | Host    | Ip   | Função   |
   | ------- | ---- | -------- |
   | Kali    | [IP] | Attacker |
   | Windows | [IP] | Victim   |
   | Wazuh   | [IP] | SIEM     |
```
---

### Attack Flow

1. Kali inicia tentativas de autenticação.
2. O sistema alvo registra os eventos
3. Wazuh Agent coleta os eventos.
4. Wazuh manager processa os eventos
5. A regra de detecção identifica o comportamento
6. Um alerta é gerado.
7. O actice Response é acionado
8. O IP atacante é bloqueado.
9. O Analista valida a resposta
10. O incidente é documentado


### Detection Threshold

*** Condição ***
5 tentativas de autenticação malsucedidas

*** Janela de Tempo ***
2 minutos

*** Resposta *** 

*** Bloqueio temporário do IP atacante. ***

    - Os valores acima serão validados e ajustados durante a implementação do laboratório..