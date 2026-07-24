# 05 - Instalação do Docker e Docker Compose

## Objetivo

Instalar e configurar o motor de conteinerização **Docker Engine** e o plugin **Docker Compose** em ambiente Linux Ubuntu Server na VPS Hostinger, estabelecendo as permissões de execução para o usuário não-root e habilitando a inicialização automática do daemon junto com o sistema operacional.

---

# Motivação

A arquitetura planejada para o servidor exige a execução de orquestradores de containers e rotinas de automação (como Easypanel, n8n, Evolution API e bancos de dados isolados).

O uso do Docker permite isolar cada aplicação em seu próprio ambiente executável, evitando conflitos de dependências do sistema operacional e facilitando a implantação rápida com máxima segurança. Além disso, a configuração adequada do grupo de usuários evita a necessidade de utilizar o superusuário (`root`) para gerenciar containers no dia a dia.

---

# Tecnologias utilizadas

- **Sistema Operacional:** Ubuntu Server (Linux x86_64)
- **Motor de Containers:** Docker Engine v29.6.2
- **Orquestração:** Docker Compose Plugin
- **Gerenciador de Serviços:** systemd
- **Linguagem/Ferramentas do Sistema:** Bash, curl, apt

---

# Passo a passo

## 1. Limpeza preventiva de versões antigas

Executada a tentativa de remoção de eventuais instalações anteriores ou pacotes nativos conflitantes fornecidos pelo repositório padrão do Ubuntu.

```bash
sudo apt remove docker docker-engine docker.io containerd runc 2>/dev/null || true
```

### Resultado e Validação

```text
Reading state information... Done
Package 'docker' is not installed, so not removed
```

### Objetivo

Garantir que não haja conflitos de bibliotecas ou binários pré-existentes antes de adicionar o repositório oficial da Docker Inc.

---

## 2. Download do script oficial de instalação

Transferência do script de instalação mantido pela comunidade oficial do Docker via transferência segura HTTPS.

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
```

### Objetivo

Obter os instaladores automatizados e validados que configuram as chaves GPG, repositórios de pacotes seguros e dependências necessárias.

---

## 3. Execução do instalador oficial

Execução do script para download e instalação do Docker Engine, containerd, runc e o plugin Docker Compose.

```bash
sudo sh get-docker.sh
```

### Resultado e Validação

```text
# Executing docker install script, commit: 5ce20f2eef3615d08fea941eda5a109e949e8ebf
+ sh -c apt-get -qq update >/dev/null
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get -y -qq install ca-certificates curl
...
INFO: Docker daemon enabled and started

Client: Docker Engine - Community
 Version:           29.6.2
 API version:       1.55

Server: Docker Engine - Community
 Engine:
  Version:          29.6.2
```

### Objetivo

Instalar a versão mais recente e estável do Docker Engine diretamente da fonte primária e inicializar o daemon do Docker via `systemd`.

---

## 4. Remoção do script temporário

Exclusão do script de instalação para manutenção da limpeza do sistema de arquivos.

```bash
rm get-docker.sh
```

### Objetivo

Boas práticas de administração de sistemas, evitando o acúmulo de arquivos temporários após o uso.

---

## 5. Configuração de permissões de usuário (Não-Root)

Inclusão do usuário não-root `edujr` no grupo do sistema `docker` para permitir o gerenciamento de containers sem uso do comando `sudo`.

```bash
sudo usermod -aG docker $USER
```

### Objetivo

Conceder acesso de leitura/escrita ao socket Unix do Docker (`/var/run/docker.sock`) para o usuário de trabalho, mantendo o princípio do menor privilégio sem necessidade de elevação para superusuário a cada comando.

---

## 6. Atualização do grupo na sessão ativa

Atualização dos privilégios de grupo da sessão atual do shell sem necessidade de desconectar e reconectar o terminal.

```bash
newgrp docker
```

### Objetivo

Recarregar a tabela de grupos do usuário na sessão Bash ativa imediatamente após a modificação.

---

## 7. Validação da instalação e execução de container de teste

Execução de um container de teste oficial para validar o funcionamento do ambiente Docker sem privilégios de superusuário.

```bash
docker run hello-world
```

### Resultado Confirmado

```text
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

### Objetivo

Confirmar que o daemon do Docker está rodando, que a imagem foi baixada do Docker Hub com sucesso e que o container foi instanciado e executado sem erros de permissão.

---

# Como validar a configuração

As seguintes verificações foram realizadas:

- [x] Pacote do Docker Engine instalado na versão estável (29.6.2)
- [x] Daemon do Docker ativado e habilitado para boot automático via `systemd`
- [x] Usuário `edujr` adicionado com sucesso ao grupo `docker`
- [x] Execução de containers sem a necessidade de `sudo` validada
- [x] Script temporário de instalação removido

---

# Problemas encontrados e soluções

Durante a verificação inicial de pacotes desatualizados, o gerenciador de pacotes reportou que o pacote `docker` não estava instalado. A mensagem foi interpretada corretamente como uma condição normal do sistema (sistema limpo), prosseguindo-se normalmente com a execução do script oficial sem necessidade de intervenção adicional.

---

# Lições aprendidas

- **Instalação via Script Oficial:** Utilizar a fonte oficial do Docker garante pacotes atualizados e inclui suporte nativo ao `docker compose` como plugin do CLI.
- **Gerenciamento de Grupo de Usuários:** A adição do usuário ao grupo `docker` elimina o risco operacional de executar comandos do dia a dia como `root`.
- **Atualização Instantânea de Sessão:** O comando `newgrp docker` evita a perda de tempo com reconexões SSH ao atualizar as permissões do shell em tempo real.
- **Rede e Firewalls com Docker:** É importante atentar-se que publicações diretas de porta no Docker (ex: `-p 8080:8080`) interagem diretamente com o `iptables`, devendo as conexões externas ser intermediadas prioritariamente por proxies reversos (Easypanel/Traefik).

---

> **📌 REGISTRO FORMAL DE IMPLANTAÇÃO**  
> **Servidor:** `srv1837565` | **Usuário Responsável:** `edujr`  
> **Data e Horário da Implantação:** 24 de Julho de 2026 às 17:16:28 (BRT - Horário de Brasília)
