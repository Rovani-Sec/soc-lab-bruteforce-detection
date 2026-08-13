# Arquitetura do Laboratório

## Visão Geral

Este laboratório foi desenvolvido para simular um cenário controlado de tentativa de força bruta contra um endpoint Windows e demonstrar o processo de detecção utilizando o Wazuh.

O ambiente é composto por máquinas virtuais isoladas em uma rede de laboratório.

---

## Componentes

| Componente | Função |
|---|---|
| Kali Linux | Simulação do atacante |
| Windows 10 | Endpoint monitorado |
| Wazuh Agent | Coleta de eventos do Windows |
| Wazuh Manager | Processamento e correlação dos eventos |
| Wazuh Dashboard | Visualização e investigação dos alertas |
| Sysmon | Telemetria adicional do endpoint |

---

## Rede do Laboratório

A comunicação entre Kali Linux e Windows utiliza uma rede **Host-Only** para permitir a simulação controlada de tráfego entre as máquinas virtuais.

### Kali Linux

```text
IP: 192.168.56.103
Interface: eth1
Função: Atacante
```
### Windows 10 
```text
IP: 192.168.56.104
Interface: Host-Only
Função: Endpoint monitorado
```
### Os endereços acima pertencem à rede privada utilizada exclusivamente pelo laboratório.

### Fluxo de Comunicação

```text
┌─────────────────────┐
│     Kali Linux      │
│  192.168.56.103     │
│                     │
│  Nmap / NetExec     │
└──────────┬──────────┘
           │
           │ Tentativas de autenticação
           ▼
┌─────────────────────┐
│     Windows 10      │
│  192.168.56.104     │
│                     │
│  Security Events    │
│  Wazuh Agent        │
│  Sysmon             │
└──────────┬──────────┘
           │
           │ Eventos de segurança
           ▼
┌─────────────────────┐
│   Wazuh Manager     │
│                     │
│  Regras / Correlação│
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Wazuh Dashboard    │
│                     │
│      Alertas        │
└─────────────────────┘
```
---

### Serviços Monitorados

Durante a preparação do laboratório foram utilizados serviços de rede do Windows para gerar telemetria de autenticação.

Entre eles:

SMB;
RDP;
SSH.

O cenário documentado neste projeto utiliza principalmente eventos de autenticação relacionados ao SMB, com o objetivo de gerar eventos 4625 no Windows.

---

### Telemetria

O Windows fornece os eventos de segurança utilizados na detecção.

O principal evento analisado neste laboratório é:

Event ID: 4625
Description: An account failed to log on

### Os campos utilizados na investigação incluem:
```text
TargetUserName
LogonType
AuthenticationPackageName
IpAddress
IpPort
Status
SubStatus
 
```

Objetivo da Arquitetura

A arquitetura permite demonstrar o fluxo completo:

```text
Ataque simulado
      ↓
Telemetria Windows
      ↓
Wazuh Agent
      ↓
Wazuh Manager
      ↓
Correlação de eventos
      ↓
Regra personalizada
      ↓
Alerta de segurança
      ↓
Investigação

```
O ambiente foi projetado para fins educacionais e de demonstração de técnicas de monitoramento, detecção e investigação de eventos de segurança.



