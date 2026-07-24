# 03 - Fail2ban

## Objetivo

Implementar proteção automatizada contra tentativas de força bruta no serviço SSH, banindo temporariamente endereços IP que excedam um número definido de tentativas de autenticação falhas em um intervalo de tempo determinado.

---

## Motivação

Mesmo com autenticação por senha desabilitada (etapa 01) e o firewall restringindo o tráfego às portas necessárias (etapa 02), a porta SSH continua publicamente acessível e recebendo tentativas de conexão constantes de scanners e bots automatizados. Essas tentativas:

- geram ruído constante nos logs do sistema, dificultando a identificação de eventos relevantes;
- consomem recursos do servidor (CPU, processos de autenticação);
- representam, em cenários de reativação temporária de senha (manutenção emergencial, por exemplo), uma superfície de ataque válida.

O Fail2ban monitora os logs de autenticação e bane automaticamente, por um período configurável, qualquer IP que exceda o limite de tentativas malsucedidas — reduzindo o impacto desse tráfego indesejado sem intervenção manual.

---

## Tecnologias utilizadas

- Fail2ban
- iptables (utilizado internamente pelo Fail2ban para aplicar os banimentos)

---

## Passo a passo

### 1. Instalação do Fail2ban

```bash
sudo apt install fail2ban -y
```

**Objetivo:** instalar o Fail2ban e suas dependências (`python3-pyasyncore`, `python3-pyinotify`, `python3-systemd`, `whois`).

---

### 2. Criação do arquivo de configuração local

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

**Objetivo:** o arquivo `jail.conf` é o modelo padrão do pacote e não deve ser editado diretamente, pois futuras atualizações do Fail2ban podem sobrescrevê-lo. O arquivo `jail.local` é uma cópia com precedência sobre o modelo, destinada a customizações.

---

### 3. Localização da seção de configuração do SSH

Em vez de buscar a seção `[sshd]` diretamente dentro do editor de texto, a linha foi localizada previamente via terminal:

```bash
grep -n "\[sshd\]" /etc/fail2ban/jail.local
```

Saída obtida:

```text
23:# [sshd]
274:[sshd]
```

A linha 23 corresponde a uma referência comentada no cabeçalho do arquivo; a seção de configuração real está na linha 274.

---

### 4. Edição da seção `[sshd]`

O arquivo foi aberto diretamente na linha identificada:

```bash
sudo nano +274 /etc/fail2ban/jail.local
```

Foram adicionadas as seguintes diretivas, logo após as linhas existentes (`port`, `logpath`, `backend`) da seção `[sshd]`:

```text
enabled = true
maxretry = 4
bantime = 3600
findtime = 600
```

**Significado de cada diretiva:**

| Diretiva | Valor | Função |
|---|---|---|
| `enabled` | `true` | Ativa a proteção para o serviço SSH (desabilitada por padrão no arquivo modelo) |
| `maxretry` | `4` | Número de tentativas de autenticação falhas antes do banimento |
| `bantime` | `3600` | Duração do banimento, em segundos (1 hora) |
| `findtime` | `600` | Janela de tempo, em segundos (10 minutos), na qual as tentativas falhas são contabilizadas |

---

### 5. Verificação do conteúdo salvo

```bash
sudo sed -n '274,290p' /etc/fail2ban/jail.local
```

**Objetivo:** conferir o conteúdo da seção editada sem reabrir o editor de texto, validando que as diretivas foram gravadas corretamente antes de reiniciar o serviço.

---

### 6. Reinicialização do serviço

```bash
sudo systemctl restart fail2ban
```

---

## Problemas encontrados e soluções

**1. Atalho `Ctrl+W` interceptado pelo navegador.**
A edição foi realizada através do terminal web embutido no painel da Hostinger, executado dentro do navegador. Ao tentar usar `Ctrl+W` (atalho de busca do nano) para localizar a seção `[sshd]`, o navegador interceptou o atalho como comando de "fechar aba", fechando a sessão em vez de abrir a busca do editor.

**Solução:** a busca pela linha foi feita previamente via `grep -n`, fora do editor, e o nano foi aberto diretamente na linha correta utilizando o parâmetro `+<número_da_linha>` (`nano +274 arquivo`), eliminando a necessidade de busca interna no editor.

**2. Texto acidental inserido no arquivo durante tentativa de busca.**
Antes da solução acima ser adotada, uma tentativa de digitar um comando `grep` dentro do próprio editor (por engano) resultou em texto acidental sendo inserido no conteúdo do arquivo.

**Solução:** o editor foi fechado sem salvar (`Ctrl+X`, seguido de `N` na confirmação "Save modified buffer?"), preservando o arquivo original, e a edição foi refeita corretamente a partir do zero.

**3. Verificação inicial com intervalo de linhas insuficiente.**
A primeira tentativa de conferir o conteúdo salvo (`sed -n '274,282p'`) não exibiu as diretivas adicionadas, pois a inserção de 4 novas linhas deslocou o conteúdo para além do intervalo consultado.

**Solução:** o intervalo de verificação foi ampliado (`sed -n '274,290p'`), exibindo a seção completa, incluindo as diretivas adicionadas.

---

## Como validar a configuração

```bash
sudo fail2ban-client status sshd
```

Saída obtida:

```text
Status for the jail: sshd
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     0
|  `- File list:        /var/log/auth.log
`- Actions
   |- Currently banned:  0
   |- Total banned:      0
   `- Banned IP list:
```

Itens confirmados:

- ✅ Fail2ban instalado e serviço ativo
- ✅ Jail `sshd` habilitada e monitorando `/var/log/auth.log`
- ✅ Parâmetros de banimento (`maxretry`, `bantime`, `findtime`) aplicados corretamente
- ✅ Nenhum erro ao reiniciar o serviço

O funcionamento efetivo do banimento (IP entrando na lista `Banned IP list`) só é observável organicamente, à medida que tentativas de autenticação falhas ocorrerem — não foi simulado neste momento para evitar autobanimento acidental.

---

## Lições aprendidas

- Terminais web (executados dentro do navegador) podem interceptar atalhos de teclado padrão de editores de texto (como `Ctrl+W`), exigindo estratégias alternativas de navegação.
- Buscar a linha de interesse via `grep -n` antes de abrir o editor, e abrir o editor diretamente na linha desejada (`nano +N arquivo`), é mais confiável do que depender de busca interna em ambientes de terminal não convencionais.
- Após qualquer edição incerta, é preferível sair do editor sem salvar e recomeçar do zero do que tentar corrigir manualmente um conteúdo já corrompido.
- Verificações de conteúdo via `sed -n` devem considerar que a inserção de novas linhas desloca a numeração do restante do arquivo.
- O Fail2ban depende diretamente do log de autenticação do sistema (`/var/log/auth.log`); qualquer alteração na localização ou rotação desse arquivo deve ser refletida na configuração da jail.
