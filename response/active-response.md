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

[captura acionamento da regra no agent windows](../screenshots/009-regra-active-response-acionada-windows)
