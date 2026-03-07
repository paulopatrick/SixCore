Treinamento de Segurança - Configuração de um firewall MIkrotik 

Equipamento: Mikrotik hPA lite (RouterOS 6.49.19 - long-term)

As regras criadas até então foram baseadas nas aulas do treinamento SixCore com o instrutor Leonardo Vieira de Albuquerque (http://linkedin.com/in/albuquerqueleonardo/).
Poderão ser adicionadas novas regras futuramente com meu melhor desenvolvimento sobre o tema - Segurança.
A sequência das regras segue sem filtro específico, ou seja, as chains estão agrupadas. 

>>> Regra 1 - FastTrack <<<

Função: Acelerar o processamento de pacotes cuja conexão já está estabelecida, pulando assim etapas de firewall e reduzindo o uso de CPU do roteador.
# Chain: forward
# Connection state: established related
# Action: fasttrack connection

>>> Regra 2 - Bloquear conexões inválidas <<<

Função: Bloqueia pacotes que fogem do padrão de conexões, exemplificando pacote que chega fora de ordem (um ACK isolado) ou um pertencente à uma conexão expirada.
# Chain: forward
# Connection state: invalid
# Action: drop

>>> Regra 3 - Bloquear conexões inválidas <<<

# Chain: input
# Connection state: invalid
# Action: drop

>>> Regra 4 - Aceita conexões estabelecidas e relacionadas <<<

Função: Aquelas conexões já conhecidas pelo roteador são liberadas para manter uma conexão ativa. Conexões novas (primeiro contato) serão validadas nas regras seguintes e, caso aprovadas, passam para o estado de conexão estabelecido e relacionado.

# Chain: input
# Connection state: established related
# Action: accept

>>> Regra 5 - Aceita conexões estabelecidas e relacionadas <<<

# Chain: forward
# Connection state: established related
# Action: accept

>>> Regra 6 - Obtém IP origem de quem tenta acesso WAN -> LAN <<<

Função: Aqueles na internet que tentam acesso direto à rede local têm seus endereços IP capturados e inseridos em uma lista "Hackers" por 7 dias.
# Chain: forward
# In. interface list: WAN
# Out. interface list: LAN
# Action: add src to address list
# Address List: Hackers
# Timeout: 7d 00:00:00

>>> Regra 7 - Obtém IP origem de quem tenta acesso externo ao roteador <<<

Função: Aqueles na internet que tentam acesso direto às portas sensíveis do roteador têm seus endereços IP capturados e inseridos em uma lista "Hackers" por 7 dias.
# Chain: input
# Protocolo: TCP
# Src. Port: 0-1024, 3389, 5900
# In. interface list: WAN
# Action: add src to address list
# Address List: Hackers
# Timeout: 7d 00:00:00

>>> Regra 8 - Obtenção de IP de origem daqueles que tentam PortScan <<<

Função: Portas baixas (0-1024) recebem um peso 3, enquanto as demais portas ditas altas um peso 1; caso alguém tente bater nessas portas de tal forma que a soma dos pesos atinja 21 em menos de 3 segundos, o endereço IP é capturado e alocado na lista "Hackers" por 7 dias.

# CHain: input
# Protocolo: TCP
# PSD (PortScan Detective): 21 | 00:00:03 | 3 | 1
# Action: add src to address list
# Address List: Hackers
# Timeout: 7d 00:00:00

>>> Regra 9 - SSH Brute Force - Estagio 0 <<<

Função: Aqueles na internet que tentarem acesso por SSH ao roteador pela porta 22 3 vezes em um intervalo inferior à 1 minuto entre as tentativas terão seus endereços IP alocados em uma lista de "Hackers" por 7 dias.
# Chain: input
# Protocolo: TCP
# DST. Port: 22
# In. Interface List: WAN
# Connection state: new
# Action: add src to address list
# Address List: SSH_estagio_1
# Timeout: 00:01:00

>>> Regra 10 - SSH Brute Force - Estagio 1 <<<

# Chain: input
# Protocolo: TCP
# DST. Port: 22
# In. Interface List: WAN
# Connection state: new
# Src. Address List: SSH_estagio_1
# Action: add src to address list
# Address List: SSH_estagio_2
# Timeout: 00:01:00

>>> Regra 11 - SSH Brute Force - Estagio 2 <<<

# Chain: input
# Protocolo: TCP
# DST. Port: 22
# In. Interface List: WAN
# Connection state: new
# Src. Address List: SSH_estagio_2
# Action: add src to address list
# Address List: Hackers
# Timeout: 7d 00:00:00

>>> Regra 12 - Sequencia de portas - Fase 0 <<<

Função: Uma medida de validar quem tem acesso às informações privilegiadas através de uma senha, um métodos conhecido como Port Knock. Quem souber a sequência correta das portas obtém acesso à lista de IPs liberados.
# Chain: input
# Protocolo: TCP
# Dst. Port: 57153
# Action: add src to address list
# Address List: Sequencia de Portas - Fase 1
# Timeout: 00:00:10

>>> Regra 13 - Sequencia de portas - Fase 1 <<<
# Chain: input
# Protocolo: TCP
# Dst. Port: 32570
# Src. Address List: Sequencia de Portas - Fase 1
# Action: add src to address list
# Address List: Sequencia de Portas - Fase 2
# Timeout: 00:00:10

>>> Regra 14 - Sequencia de portas - Fase 2 <<<
# Chain: input
# Protocolo: UDP
# Dst. Port: 4851
# Src. Address List: Sequencia de Portas - Fase 2
# Action: add src to address list
# Address List: Sequencia de Portas - Fase 3
# Timeout: 00:00:10

>>> Regra 15 - Sequencia de portas - Fase 3 <<<
# Chain: input
# Protocolo: TCP
# Dst. Port: 49001
# Src. Address List: Sequencia de Portas - Fase 3
# Action: add src to address list
# Address List: Sequencia de Portas - Fase 4
# Timeout: 00:00:10

>>> Regra 16 - Sequencia de portas - Fase 4 <<<
# Chain: input
# Protocolo: TCP
# Dst. Port: 61013
# Src. Address List: Sequencia de Portas - Fase 4
# Action: add src to address list
# Address List: Sequencia de Portas - Fase 5
# Timeout: 00:00:10

>>> Regra 17 - Sequencia de portas - Fase 5 <<<
# Chain: input
# Protocolo: TCP
# Dst. Port: 26012
# Src. Address List: Sequencia de Portas - Fase 5
# Action: add src to address list
# Address List: Sequencia de Portas - Liberados
# Timeout: 04:00:00

>>> Regra 18 - Permissao de acesso a internet <<<
# Chain: forward
# In. Interface LIst: LAN
# Out. Interface List: WAN
# Action: Accept

>>> Regra 19 - Permite acesso LAN ao roteador (exceto às portas administrativas de rede)
# Chain: input
# Protocolo: TCP
# Dst. Ports: !3596, 2498, 5124 (portas subtitutas às 8129, 80, 443)
# In. Interface List: LAN
# Action: Accept

>>> Regra 20 - Libera WinBox, Webfig <<< 
# Chain: input
# Protocolo: TCP
# Dst. Ports: 3596, 2498, 5124
# In. Interface List: LAN
# Src. Address List: Sequencia de Portas - Liberados 
# Action: Accept

CONTINUA....








