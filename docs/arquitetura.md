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

3. Reconhecimento

Antes da simulação das autenticações, foi realizado reconhecimento do endpoint utilizando Nmap.

O objetivo foi identificar serviços de rede disponíveis e determinar quais serviços poderiam ser utilizados durante o teste.

A enumeração identificou o serviço SMB no endpoint Windows.

A evidência do reconhecimento está disponível em:

[Ver Evidência - Nmap SMB Enumeration](.../screenshots/001-nmap-smb-enumeration.png)
