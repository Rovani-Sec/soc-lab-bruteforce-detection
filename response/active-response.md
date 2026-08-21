# 🛡️ Detecção e Resposta Ativa (Active Response) com Wazuh

## 🎯 Objetivo
Este laboratório demonstra a capacidade de resposta automatizada a incidentes (DFIR) utilizando o Wazuh XDR. O objetivo foi detectar um ataque de força bruta via protocolo SMB e acionar um bloqueio imediato no Firewall do Windows, isolando o IP do atacante (Kali Linux).

## 🛠️ Tecnologias e Ambiente
* **SIEM / XDR:** Wazuh Manager v4.14.6 (Linux)
* **Atacante:** Kali Linux (Utilizando `NetExec / nxc` para Brute Force)
* **Vítima:** Windows 10 Pro (com Wazuh Agent)
* **Tática MITRE ATT&CK:** [T1110 - Brute Force](https://attack.mitre.org/techniques/T1110/)

## ⚙️ Configuração da Regra de Detecção


Foi criada uma regra customizada (`100100`) para monitorar falhas consecutivas de logon (Event ID 4625), disparando um alerta de nível crítico caso ocorram 5 tentativas em menos de 60 segundos oriundas do mesmo IP.

```xml
<group name="windows,authentication,bruteforce,">
  <rule id="100100" level="12" frequency="5" timeframe="60">
    <if_matched_sid>60122</if_matched_sid>
    <same_field>win.eventdata.ipAddress</same_field>
    <description>Windows SMB Brute Force: Multiple failed logons from $(win.eventdata.ipAddress)</description>
    <mitre>
      <id>T1110</id>
    </mitre>
  </rule>
</group>
```

## Resposta Ativa (Active Response)

Para conter a ameaça, o Wazuh Manager foi configurado para executar o binário netsh.exe no agente Windows afetado, criando dinamicamente uma regra de bloqueio (Drop) no Windows Defender Firewall contra o IP agressor.

Comando configurado no Manager (ossec.conf):

No manager do wazuh foi utilizado o comando:

```text
<command>
  <name>win-netsh-block</name>
  <executable>netsh.exe</executable>
  <timeout_allowed>yes</timeout_allowed>
</command>

<active-response>
  <command>win-netsh-block</command>
  <location>defined-agent</location>
  <agent_id>001</agent_id>
  <rules_id>100100</rules_id>
  <timeout>300</timeout>
</active-response>
```
Evidências de Execução

Log de execução capturado no Windows (active-responses.log), comprovando que o script foi iniciado, recebeu o IP malicioso via payload JSON e executou o bloqueio com sucesso:

<img width="1285" height="174" alt="009-regra-active-response-acionada-windows" src="https://github.com/user-attachments/assets/1f209b38-7a53-4ab2-b335-d2f37f107edd" />

## Bloqueio do IP Atacante

Depois que o ataque é realizado contra o host windows [ataque contra o host[(../sreenshots/002-Kali-attack.png)

<img width="1404" height="375" alt="011-ping-packet-loss" src="https://github.com/user-attachments/assets/e0e13d34-2370-4d96-be86-505c19fbc246" />


