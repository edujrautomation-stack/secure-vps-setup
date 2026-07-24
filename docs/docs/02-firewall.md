02 - Firewall (UFW)
Objetivo
Restringir o tráfego de rede recebido pela VPS, permitindo apenas as conexões estritamente necessárias para administração remota e para o funcionamento das aplicações web que serão hospedadas. Ao final desta etapa, a VPS aceitará conexões de entrada apenas nas portas 22 (SSH), 80 (HTTP) e 443 (HTTPS), rejeitando qualquer outra tentativa de conexão por padrão.


Motivação
Um servidor exposto à internet sem firewall aceita tentativas de conexão em qualquer porta disponível. Isso amplia a superfície de ataque, pois:

serviços rodando em portas não previstas ficam acessíveis publicamente sem necessidade;
scanners automatizados varrem a internet constantemente em busca de portas abertas;
cada porta aberta desnecessariamente é um vetor adicional de ataque.

A adoção de uma política de negação por padrão (deny by default), liberando apenas o necessário, reduz significativamente essa superfície.


Tecnologias utilizadas
UFW (Uncomplicated Firewall) — interface simplificada para o iptables/nftables do Linux


Passo a passo
1. Verificação da instalação existente
O Ubuntu Server já inclui o UFW pré-instalado, porém desativado por padrão.

sudo ufw status

Saída obtida:

Status: inactive


2. Definição da política padrão de entrada
sudo ufw default deny incoming

Objetivo: estabelecer que toda conexão de entrada seja recusada por padrão, exigindo liberação explícita para cada porta necessária.


3. Definição da política padrão de saída
sudo ufw default allow outgoing

Objetivo: permitir que a própria VPS continue iniciando conexões para fora (atualizações de pacotes, chamadas de API, etc.), já que a restrição se aplica apenas ao tráfego de entrada.


4. Liberação da porta de administração remota (SSH)
sudo ufw allow OpenSSH

Objetivo: garantir a continuidade do acesso administrativo via SSH (porta 22).

Atenção: esta liberação deve ocorrer obrigatoriamente antes da ativação do firewall (passo 7). Ativar o UFW sem liberar a porta SSH previamente resulta na perda de acesso remoto ao servidor.


5. Liberação das portas HTTP e HTTPS
sudo ufw allow 80/tcp

sudo ufw allow 443/tcp

Objetivo: preparar a VPS para expor aplicações web (EasyPanel, n8n, Evolution API) que serão configuradas em etapas posteriores, via HTTP (80) e HTTPS (443).


6. Conferência das regras antes da ativação
sudo ufw show added

Saída obtida:

Added user rules (see 'ufw status' for running firewall):

ufw allow OpenSSH

ufw allow 80/tcp

ufw allow 443/tcp

Objetivo: validar que todas as regras planejadas foram registradas corretamente antes de ativar o firewall, evitando bloqueio acidental de portas necessárias.


7. Ativação do firewall
sudo ufw enable

O sistema exibe um aviso padrão informando que a operação pode interromper conexões SSH ativas. Como a porta SSH já havia sido liberada no passo 4, a operação foi confirmada com segurança.


Problemas encontrados e soluções
1. Comandos colados sem separação executados incorretamente. Ao colar dois comandos consecutivos (ufw allow 80/tcp e ufw allow 443/tcp) sem quebra de linha entre eles, apenas o primeiro foi executado como comando independente. A conferência com ufw show added (passo 6) permitiu identificar que a regra da porta 443 não havia sido criada, sendo corrigida com uma nova execução isolada do comando.

Lição prática: sempre validar as regras com ufw show added antes de ativar o firewall — evita descobrir uma porta faltante somente depois da ativação, quando a correção exigiria uma nova regra em produção.


Como validar a configuração
sudo ufw status verbose

Saída obtida:

Status: active

Logging: on (low)

Default: deny (incoming), allow (outgoing), disabled (routed)

New profiles: skip

To                         Action      From

--                         ------      ----

22/tcp (OpenSSH)           ALLOW IN    Anywhere

80/tcp                     ALLOW IN    Anywhere

443/tcp                    ALLOW IN    Anywhere

22/tcp (OpenSSH (v6))      ALLOW IN    Anywhere (v6)

80/tcp (v6)                ALLOW IN    Anywhere (v6)

443/tcp (v6)               ALLOW IN    Anywhere (v6)

Itens confirmados:

✅ Firewall ativo e configurado para iniciar automaticamente com o sistema
✅ Política padrão de entrada: negar
✅ Política padrão de saída: permitir
✅ Portas 22, 80 e 443 liberadas para IPv4 e IPv6
✅ Acesso SSH mantido após a ativação


Lições aprendidas
A porta de administração remota deve sempre ser liberada antes da ativação do firewall.
Comandos colados em sequência no terminal podem não ser executados como esperado; cada comando deve ser confirmado individualmente quando há dúvida.
ufw show added é uma etapa de verificação intermediária útil, pois permite revisar as regras planejadas antes de aplicá-las de fato com ufw enable.
O UFW aplica automaticamente as regras tanto para IPv4 quanto para IPv6 ao liberar uma porta.
Uma política de negação por padrão, com liberação explícita apenas do necessário, reduz drasticamente a superfície de ataque de um servidor exposto à internet.

✅ Concluído

Data da configuração:

23/07/2026
