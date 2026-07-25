# 06 - Instalação e Configuração do Easypanel com Docker Swarm

## Objetivo

Instalar, configurar e estabilizar o orquestrador de aplicações **Easypanel** e o proxy reverso **Traefik**, rodando sobre o modo **Docker Swarm**, na VPS Ubuntu Server (`srv1837565`), resolvendo falhas de inicialização e um loop de erro no backend causado por uma tentativa de instalação manual anterior.

---

## Motivação

Com o Docker Engine já operacional na VPS, tornou-se necessária uma plataforma de gerenciamento visual para orquestrar containers, bancos de dados, fluxos do n8n e a Evolution API sem depender exclusivamente de comandos manuais do Docker CLI.

O Easypanel foi escolhido por integrar nativamente o Docker Swarm (orquestração) e o Traefik (proxy reverso com emissão automática de certificados SSL/TLS via Let's Encrypt), permitindo provisionar novos serviços rapidamente, com HTTPS automático, através de uma interface web.

---

## Tecnologias utilizadas

- Ubuntu Server (Linux x86_64) — host `srv1837565`
- Docker Swarm (modo single-node manager)
- Easypanel v1.10.3
- Traefik v3.6.7
- Docker CLI / Docker API v1.55

---

## Passo a passo

### 1. Inicialização do cluster Docker Swarm

```bash
sudo docker swarm init
```

**Objetivo:** habilitar o modo Swarm no nó local, pré-requisito para que o Easypanel e o Traefik rodem como serviços gerenciados (`docker service`), com rede overlay isolada.

Saída obtida:

```text
Swarm initialized: current node (k2qh7jwg5md8) is now a manager.

To add a worker to this swarm, run the following command:
    docker swarm join --token SWMTKN-1-... 179.197.228.49:2377
```

---

### 2. Remoção de tentativas de instalação anteriores

Antes da instalação automatizada, houve uma tentativa manual de criar o serviço via `docker service create`, que resultou em arquivos de configuração incompletos. Esses resíduos foram removidos para evitar conflito com a instalação oficial:

```bash
docker service rm easypanel 2>/dev/null || true
sudo rm -rf /etc/easypanel
sudo mkdir -p /etc/easypanel
```

**Objetivo:** eliminar estado parcial e arquivos de configuração corrompidos que causavam exceções no backend da aplicação durante o boot.

**Nota técnica:** o `2>/dev/null || true` garante que o comando não interrompa o script caso o serviço `easypanel` ainda não exista — evita erro desnecessário em uma tentativa de remoção "preventiva".

---

### 3. Execução do instalador oficial

```bash
curl -sSL https://get.easypanel.io -o install.sh && sudo bash install.sh && rm install.sh
```

**Objetivo:** utilizar o script oficial de instalação, que configura corretamente os volumes, a rede overlay e os serviços do Traefik e do Easypanel, evitando erros de parâmetros que ocorrem em criações manuais via Docker CLI.

Saída obtida (resumida):

```text
Docker already installed
latest: Pulling from easypanel/easypanel
Docker API version set to 1.55
Swarm was initilized
Network was created
Default certificate was created
3.6.7: Pulling from library/traefik
Traefik service was created
Easypanel service was created

Easypanel was installed successfully on your server!

    http://179.197.228.49:3000
```

---

### 4. Validação dos serviços no Swarm

```bash
docker service ls
```

**Objetivo:** confirmar que ambos os serviços (Easypanel e Traefik) atingiram o status de réplica saudável (`1/1`).

Saída obtida:

```text
ID             NAME                MODE         REPLICAS   IMAGE                        PORTS
84g4aq1devnu   easypanel           replicated   1/1        easypanel/easypanel:latest   *:3000->3000/tcp
k83rogipo4so   easypanel-traefik   replicated   1/1        traefik:3.6.7                *:80->80/tcp, *:443->443/tcp
```

---

## Problemas encontrados e soluções

**1. Loop de reinicialização no backend (`TypeError: Cannot read properties of undefined (reading 'email')`).**

**Causa raiz:** uma tentativa manual anterior de criar o serviço via `docker service create` deixou arquivos de configuração incompletos em `/etc/easypanel`, sem o objeto de e-mail do usuário administrador esperado pela aplicação no boot.

**Solução:** remoção do serviço (`docker service rm easypanel`) e limpeza completa do diretório de persistência (`sudo rm -rf /etc/easypanel`) antes de rodar o instalador oficial do zero.

**2. Erro de sintaxe ao baixar o script de instalação (`<!doctype html>`).**

**Causa raiz:** uso de uma URL legada (`https://easypanel.io/get.sh`), que redirecionava para a página institucional em HTML da plataforma em vez do script `.sh`, resultando no erro `install.sh: line 1: syntax error near unexpected token 'newline'` ao tentar executar HTML como Bash.

**Solução:** uso do endpoint de download direto e oficial, `https://get.easypanel.io`.

**3. Erros de parâmetro em tentativas manuais de criação do serviço (`unknown flag: --replica-deprecate-control`, `unknown flag: --e`).**

**Causa raiz:** sintaxe incorreta ao tentar criar o serviço manualmente via Docker CLI, sem seguir a configuração exata esperada pelo Easypanel (variáveis de ambiente, volumes e labels do Traefik).

**Solução:** abandono da abordagem manual em favor do script de instalação oficial, que já define esses parâmetros corretamente.

---

## Como validar a configuração

- ✅ Cluster Docker Swarm ativo, em modo manager
- ✅ Diretório `/etc/easypanel` limpo e recriado com permissões corretas antes da instalação
- ✅ Serviço do Traefik (v3.6.7) ativo, escutando nas portas 80 e 443
- ✅ Serviço do Easypanel ativo, com 1/1 réplica operacional na porta 3000
- ✅ Script de instalação temporário (`install.sh`) removido após uso
- ✅ Painel web acessível via `http://179.197.228.49:3000`

---

## Lições aprendidas

- Para orquestradores que dependem de proxy reverso e integração com o socket do Docker, o instalador oficial é significativamente mais confiável do que criar os serviços manualmente via Swarm — ele já resolve a combinação correta de volumes, redes e labels.
- Sempre validar o conteúdo retornado por um `curl` de instalação antes de executá-lo como script, especialmente quando a URL pode redirecionar para uma página HTML em vez do arquivo esperado.
- Quando uma aplicação Node.js falha no boot por tentar ler uma propriedade de um objeto indefinido (erro típico de estado de configuração incompleto), limpar o volume/diretório de persistência e reinstalar do zero costuma ser mais rápido do que tentar corrigir o estado manualmente.
- Docker Swarm em conjunto com Traefik simplifica a adição de novos serviços no futuro, já que o Traefik detecta automaticamente as labels dos containers para expor rotas e emitir certificados.
- Tentativas manuais de configuração que falham devem ser completamente revertidas (serviço removido, diretório de dados limpo) antes de uma nova tentativa — estado parcial residual é uma causa comum de erros difíceis de diagnosticar.

---

**Registro da implantação**

| Campo | Valor |
|---|---|
| Servidor | `srv1837565` |
| Usuário responsável | `edujr` |
| Data e horário | 24 de julho de 2026, 18:05 (horário de Brasília) |
