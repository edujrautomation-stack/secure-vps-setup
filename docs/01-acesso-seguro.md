01 - Acesso Seguro

Objetivo
Configurar um acesso seguro à VPS eliminando práticas inseguras de autenticação e adotando boas práticas de administração de servidores Linux. Ao final desta etapa, a VPS estará preparada para ser administrada exclusivamente por um usuário não-root autenticado por chave SSH.


Motivação
Uma VPS recém-criada normalmente permite login como root utilizando senha. Essa configuração representa um risco, pois:

ataques automatizados tentam acessar continuamente o usuário root;
senhas podem ser descobertas por força bruta ou vazamentos;
utilizar o usuário root para tarefas diárias aumenta o impacto de erros operacionais.

Para reduzir esses riscos foram implementadas medidas de endurecimento (hardening) do serviço SSH.


Tecnologias utilizadas
Ubuntu Server 24.04 LTS
OpenSSH Server
OpenSSH Client
SSH Key (Ed25519)
sudo
UFW (mencionado nos próximos passos, não faz parte desta etapa)


Passo a passo
1. Criação de usuário administrativo
Foi criado um usuário não-root para administração da VPS, com permissões administrativas concedidas através do grupo sudo.

adduser edujr

usermod -aG sudo edujr

Objetivo: evitar o uso diário do usuário root.


2. Geração da chave SSH Ed25519
Foi gerado, na máquina local (não na VPS), um par de chaves SSH utilizando o algoritmo Ed25519.

ssh-keygen -t ed25519 -C "email-de-referencia@exemplo.com"

A chave privada permaneceu armazenada localmente, protegida por passphrase. A chave pública foi copiada manualmente para a VPS, adicionada ao arquivo ~/.ssh/authorized_keys do usuário administrativo:

mkdir -p ~/.ssh

chmod 700 ~/.ssh

nano ~/.ssh/authorized_keys   # conteúdo colado manualmente

chmod 600 ~/.ssh/authorized_keys

chown -R edujr:edujr ~/.ssh

Objetivo: substituir autenticação por senha por autenticação baseada em criptografia assimétrica.

Nota técnica: o SSH exige permissões restritas nesses arquivos (700 para o diretório .ssh, 600 para authorized_keys) e recusa a autenticação por chave caso as permissões estejam mais abertas que o esperado, mesmo que o conteúdo esteja correto.


3. Teste de autenticação
Antes de qualquer alteração no serviço SSH, foi aberta uma segunda sessão, mantendo a sessão original ativa como medida de segurança.

ssh edujr@IP_DA_VPS

O login utilizando a chave SSH foi testado com sucesso. Somente após confirmar o funcionamento da autenticação foi iniciado o endurecimento da configuração.

Objetivo: evitar perda de acesso ao servidor.


4. Desabilitação do login do usuário root
Foi alterado o arquivo:

/etc/ssh/sshd_config

Configuração aplicada:

PermitRootLogin no

Objetivo: impedir autenticação direta utilizando o usuário root.


5. Desabilitação de autenticação por senha
Foi alterada a configuração:

PasswordAuthentication no

Objetivo: permitir acesso somente através de chaves SSH.


6. Limitação das tentativas de login
Foi configurado:

MaxAuthTries 3

Objetivo: reduzir a superfície de ataque por tentativas de força bruta durante a autenticação.


7. Reinicialização do serviço SSH
Após todas as alterações, o serviço SSH foi reiniciado:

sudo systemctl restart ssh

Nota técnica: no Ubuntu, o serviço é registrado como ssh, não sshd — diferente de outras distribuições (como CentOS/RHEL), onde o nome do serviço é sshd. Usar o nome incorreto resulta em erro Unit sshd.service not found.

Em seguida foi realizado um novo teste de acesso utilizando a chave configurada, sem fechar a sessão anterior, garantindo um caminho de retorno em caso de falha na nova configuração.


Problemas encontrados e soluções
Durante a execução desta etapa foram identificados os seguintes problemas:

1. Diretivas do sshd_config comentadas por padrão. O arquivo de configuração padrão do Ubuntu mantém diretivas como PasswordAuthentication e MaxAuthTries comentadas (prefixadas com #), com os valores padrão do OpenSSH sendo aplicados implicitamente. Editar apenas o valor sem remover o # não surte efeito. Solução: localizar e descomentar cada diretiva antes de alterar o valor, ou aplicar a alteração via sed:

sudo sed -i 's/^#\?PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config

sudo sed -i 's/^#\?PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config

sudo sed -i 's/^#\?MaxAuthTries.*/MaxAuthTries 3/' /etc/ssh/sshd_config

2. Conflito de configuração durante atualização do pacote openssh-server. Ao executar apt upgrade, o gerenciador de pacotes identificou que o sshd_config havia sido modificado manualmente e perguntou se deveria manter a versão local ou substituir pela versão padrão do pacote. Foi selecionada a opção "keep the local version currently installed", preservando as configurações de hardening já aplicadas.

3. Nome incorreto do serviço ao reiniciar. A tentativa inicial de reiniciar o serviço via systemctl restart sshd falhou, pois no Ubuntu o serviço correto é ssh. Corrigido utilizando systemctl restart ssh.


Como validar a configuração
As seguintes verificações foram realizadas:

✅ Login utilizando o usuário administrativo
✅ Login utilizando chave SSH
✅ Login do usuário root bloqueado
✅ Login por senha desabilitado
✅ Serviço SSH reiniciado sem erros

Comando utilizado para conferir as diretivas aplicadas:

sudo grep -E "^PermitRootLogin|^PasswordAuthentication|^MaxAuthTries" /etc/ssh/sshd_config

Saída esperada:

PermitRootLogin no

PasswordAuthentication no

MaxAuthTries 3

A tentativa de login com root resulta em solicitação de senha seguida de recusa permanente (Permission denied), independentemente da senha informada — comportamento esperado, já que o SSH não revela antecipadamente se um usuário está bloqueado.


Lições aprendidas
O usuário root não deve ser utilizado para administração diária.
Chaves SSH oferecem maior segurança do que autenticação por senha.
Sempre deve ser realizado um teste de acesso, em uma sessão paralela, antes de desabilitar a autenticação por senha.
Alterações incorretas no SSH podem causar perda total de acesso ao servidor.
O nome do serviço SSH (ssh vs sshd) varia entre distribuições Linux.
Diretivas comentadas no sshd_config não têm efeito até serem descomentadas.
Atualizações do sistema (apt upgrade) podem detectar configurações customizadas e solicitar confirmação antes de sobrescrevê-las.
O princípio do menor privilégio é fundamental para a segurança de servidores Linux.

✅ Concluído

Data da configuração:

23/07/2026
