# Soc Lab - Brute Force Detection & Autmate Response

  
  Laboratório de Blue Team focado na detecção, investigação e resposta automatizada a ataque de força bruta


### Objetivo

Construir um ambiente de laboratório capaz de:

 - Simular ataques de força bruta;

 - Coletar eventos de autenticação;

 - Detectar múltiplas tentativas de login malsucedidos;

 - Gerar alertas de segurança;

 - Investigar os eventos no SIEM;

 - Bloquear automaticamente o IP atacante;

 - Documentar o incidente;

 - Mapear a atividade ao Mitre ATT&CK;

  ---

### Arquitetura




### Tecnologias 

| Tecnologias  | Função                   |
| ------------ | ------------------------ |
| Wazuh        | SIEM / Detecção          |
| Wazuh Agent  | Coleta de Eventos        |
| Sysmon       | Telemetria de Endpoints  |
| Windows      | Sistema monitorado       |
| Kali Linux   | Simulação do atacante    |
| Hydra        | Simulação de Brute Force |
| PowerShell   | Automação / Investigação |
| MITRE ATT&CK | Classificação da técnica |

### Cenário

O Laboratório simula um atacante realizando múltiplas tentativas de autenticação contra um serviço exposto.

O objetivo é identificar o comportamento de força bruta e executar uma resposta automatizada.

#### Condição de detecção
 - 5 tentativas de autenticação malsucedidas provenientes do mesmo IP dentro de uma janela de tempo definida

#### Resposta

 - Após atingir o threshold configurado, o endereço IP do atacante será bloqueado automaticamente


### Processo de investigação

O fluxo será:

 1. Identificação do alerta;
 2. Identificação do endereço IP de Origem;
 3. Identificação do usuário alvo;
 4. Análise dos eventos;
 5. Construção da timeline;
 6. Verificação de possível comprometimento;
 7. Classificação do incidente;
 8. Mapeamento MITRE ATT&CK;
 9. Contenção;
 10. Documentação.

### Automated Response

O projeto também implementará uma resposta automaizada utilizando o mecanismo de ActiVE Response do Wazuh.

#### Ação

Bloqueio temporário do endereço IP identificado como origem do ataque.

#### Considerações

A implementação deverá considerar:
 - Falso Positivo;
 - IPs confiáveis;
 - Limite de tentativas;
 - Duração do bloqueio;
 - Possibilidade de reversão;
 - Auditoria da ação executada.

## Resultado

Em construção...

### Evidências 

As evidências do laboratório serão documentadas na pasta `/screeshots`.

