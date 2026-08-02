# 12 - Backup

## Objetivo

Implementar uma rotina de backup automática e testada para os dados dos clientes, combinando backup de dados via `rclone` para nuvem externa (Google Drive), snapshot de disco inteiro no provedor (Hostinger) como segunda camada, e validação real de restauração — eliminando a suposição de que um backup funciona sem nunca tê-lo restaurado.

---

# Motivação

Todas as camadas de segurança implementadas até aqui (hardening, firewall, isolamento por cliente) protegem contra ataques e configurações incorretas, mas não protegem contra falha de hardware, erro humano (ex: exclusão acidental de dados) ou indisponibilidade do provedor de hospedagem. Backup é a única defesa contra esses cenários — e um backup nunca restaurado com sucesso não pode ser considerado confiável.

---

# Tecnologias utilizadas

- rclone
- Google Cloud Console (OAuth Client próprio)
- Google Drive
- cron
- Snapshots nativos da Hostinger
- Bash / tar

---

# Passo a passo

## 1. Instalação e configuração do rclone

```bash
sudo apt install rclone -y
rclone config
```

Durante a configuração interativa, foram feitas as seguintes escolhas:

- **Tipo de armazenamento:** Google Drive
- **Client ID / Client Secret:** gerados manualmente no Google Cloud Console (ver seção seguinte), em vez de usar a chave genérica compartilhada do rclone
- **Escopo (scope):** `drive.file` — acesso restrito apenas a arquivos criados pelo próprio rclone, sem visibilidade sobre os demais arquivos do Google Drive da conta
- **Auto config:** `No` — por se tratar de uma máquina remota sem navegador (headless), a autorização foi concluída via `rclone authorize` executado em uma máquina local com navegador, colando o token gerado de volta na VPS
- **Shared Drive:** `No` — a conta utilizada é uma conta Google pessoal padrão, sem Drives Compartilhados (recurso do Google Workspace)

### Objetivo

Utilizar uma credencial OAuth própria (em vez da chave genérica do rclone) para obter melhor desempenho e maior controle sobre a integração, restringindo o escopo de acesso ao mínimo necessário (apenas arquivos que o próprio processo de backup cria).

---

## 2. Criação do Client OAuth no Google Cloud Console

1. Criado um projeto dedicado (`nexflow-dx-backup`) no Google Cloud Console
2. Ativada a **Google Drive API** para o projeto
3. Configurada a tela de consentimento OAuth (Google Auth Platform), tipo **External**
4. Adicionado o e-mail da conta utilizada como **usuário de teste** (necessário enquanto o app permanece em modo de teste, não publicado)
5. Criado um cliente OAuth do tipo **App para computador** (Desktop app), gerando Client ID e Client Secret

### Objetivo

Ter uma credencial de API isolada e sob controle próprio, evitando depender da chave genérica e compartilhada do rclone, sujeita a limites de uso compartilhados entre todos os usuários da ferramenta.

---

## 3. Validação da conexão

```bash
rclone lsd gdrive:
rclone mkdir gdrive:teste-nexflow
rclone lsd gdrive:
rclone rmdir gdrive:teste-nexflow
```

### Objetivo

Confirmar que a conexão tem permissão de leitura e escrita antes de depender dela para backups reais. A criação e remoção de uma pasta de teste validou ambas as operações.

---

## 4. Identificação do local de armazenamento dos dados

```bash
docker ps --format "table {{.Names}}\t{{.Image}}"
docker inspect <nome-do-container> --format '{{json .Mounts}}'
```

### Objetivo

Antes de escrever o script de backup, foi necessário identificar onde os dados dos serviços de fato residem em disco. A inspeção revelou que o EasyPanel utiliza *bind mounts* (não volumes Docker nomeados) para persistência de dados, armazenando-os em:

```
/etc/easypanel/projects/<cliente>/<serviço>/data
```

Essa descoberta simplificou a estratégia de backup: em vez de extrair dados via comandos Docker, os dados podem ser compactados diretamente a partir do sistema de arquivos da VPS.

---

## 5. Criação do script de backup

Criado em `/opt/backup-scripts/backup-nexflow.sh`:

```bash
#!/bin/bash
set -e

DATE=$(date +%Y-%m-%d_%H-%M)
BACKUP_DIR="/opt/backups"
SOURCE_BASE="/etc/easypanel/projects"
RCLONE_CONFIG="/home/edujr/.config/rclone/rclone.conf"

mkdir -p "$BACKUP_DIR"

for CLIENT_PATH in "$SOURCE_BASE"/*/; do
  CLIENT=$(basename "$CLIENT_PATH")
  ARCHIVE="$BACKUP_DIR/${CLIENT}_${DATE}.tar.gz"
  echo "Compactando dados do cliente $CLIENT..."
  tar czf "$ARCHIVE" -C "$SOURCE_BASE" "$CLIENT"
  echo "Enviando $ARCHIVE para o Google Drive..."
  rclone --config "$RCLONE_CONFIG" copy "$ARCHIVE" "gdrive:nexflow-dx-backups/$CLIENT" --create-empty-src-dirs
done

find "$BACKUP_DIR" -name "*.tar.gz" -mtime +3 -delete

echo "Backup concluído em $(date)"
```

```bash
sudo chmod +x /opt/backup-scripts/backup-nexflow.sh
```

### Decisões de design

- **Descoberta dinâmica de clientes:** em vez de listar os clientes manualmente em uma variável fixa, o script itera sobre as pastas existentes em `/etc/easypanel/projects/`. Isso garante que a rotina de backup inclua automaticamente qualquer cliente novo criado no futuro, sem exigir edição manual do script.
- **Retenção local limitada:** backups locais com mais de 3 dias são apagados automaticamente, evitando consumo crescente de disco na VPS — a retenção de longo prazo fica a cargo do Google Drive.
- **Configuração explícita do rclone:** o caminho do arquivo de configuração do rclone é especificado explicitamente via `--config`, evitando um problema de permissão descrito na seção seguinte.

### Objetivo

Automatizar o backup de todos os clientes com um único script reutilizável e de baixa manutenção, sem acoplamento a uma lista fixa de nomes de clientes.

---

## 6. Agendamento via cron

```bash
(sudo crontab -l 2>/dev/null; echo "0 3 * * * /opt/backup-scripts/backup-nexflow.sh >> /var/log/nexflow-backup.log 2>&1") | sudo crontab -
```

### Objetivo

Executar o backup automaticamente todos os dias às 3h da manhã (horário de baixo uso do servidor), registrando a saída do script em um arquivo de log para auditoria posterior, sem depender de execução manual.

---

## 7. Snapshot nativo da Hostinger

Verificado no painel da Hostinger (**VPS → Backups e monitoramento → Snapshots e Backups**) que o plano já inclui, por padrão, backup automático semanal do disco inteiro da VPS, sem custo adicional. A opção de upgrade para backup diário pago (R$16,99/mês) foi avaliada e não contratada neste momento — o backup semanal gratuito já atende ao objetivo de segunda camada de proteção contra falha total do servidor, complementando o backup diário de dados via rclone.

### Objetivo

Garantir uma camada de recuperação independente da aplicação — um snapshot de disco inteiro permite recuperar o servidor mesmo em caso de falha catastrófica que inviabilize a restauração seletiva de dados.

---

## 8. Teste de restauração

```bash
mkdir -p /tmp/teste-restore
rclone --config /home/edujr/.config/rclone/rclone.conf copy gdrive:nexflow-dx-backups/ultragas /tmp/teste-restore
ls -lh /tmp/teste-restore

mkdir -p /tmp/teste-restore/extraido
tar xzf /tmp/teste-restore/ultragas_*.tar.gz -C /tmp/teste-restore/extraido
ls -la /tmp/teste-restore/extraido/ultragas
ls -la /tmp/teste-restore/extraido/ultragas/ultragas-postgres/data | head -20
```

O backup do cliente `ultragas` foi baixado do Google Drive de volta para a VPS, extraído em uma pasta temporária isolada (sem sobrescrever dados em produção), e inspecionado manualmente. A estrutura interna da pasta de dados do Postgres (`PG_VERSION`, `base`, `pg_wal`, `pg_hba.conf`, entre outros) confirmou que os dados foram capturados de forma íntegra.

```bash
rm -rf /tmp/teste-restore
```

### Objetivo

Validar de ponta a ponta que um backup pode de fato ser recuperado — a única forma confiável de confirmar que a rotina de backup funciona é executando o processo de restauração, não apenas verificando que o script de backup roda sem erro.

---

# Problema encontrado e solução: configuração do rclone sob `sudo`

Ao executar o script de backup pela primeira vez com `sudo` (necessário para ler `/etc/easypanel/projects/`, pertencente ao root), o envio ao Google Drive falhou com o erro `didn't find section in config file`.

**Causa raiz:** a configuração do rclone (incluindo a autorização OAuth) foi criada enquanto conectado como o usuário `edujr`, sendo salva em `/home/edujr/.config/rclone/rclone.conf`. Ao rodar o script com `sudo`, o processo passa a ser executado como `root`, que possui seu próprio diretório de configuração (vazio, sem a autorização realizada).

**Solução aplicada:** o script foi ajustado para especificar explicitamente o caminho do arquivo de configuração do usuário `edujr` via `rclone --config "/home/edujr/.config/rclone/rclone.conf"`, independentemente de qual usuário o executa.

---

# Como validar a configuração

As seguintes verificações foram realizadas:

✅ Conexão do rclone com Google Drive validada (criação e remoção de pasta de teste)
✅ Script de backup executado com sucesso para os 3 clientes, com descoberta automática de clientes
✅ Cron configurado e confirmado via `crontab -l`
✅ Snapshot semanal gratuito da Hostinger confirmado como ativo por padrão
✅ Restauração completa testada: download do backup, extração e inspeção da integridade dos dados do Postgres

---

# Problemas encontrados e soluções

Além do problema de configuração do rclone sob `sudo` (detalhado acima), houve dificuldades recorrentes ao editar arquivos multi-linha (o script de backup e o crontab) via `nano` no terminal do PowerShell — atalhos como `Ctrl+O` e `Ctrl+X` nem sempre eram interpretados corretamente pelo terminal, resultando em edições perdidas (como uma primeira tentativa de configuração do cron que não foi salva). A solução adotada foi evitar o `nano` para esses casos, substituindo por comandos não interativos: `cat > arquivo << 'EOF' ... EOF` (ou `sudo tee` quando permissões de root eram necessárias) para criação de arquivos, e composição via pipe (`crontab -l | ... | crontab -`) para edição do cron.

---

# Lições aprendidas

- Antes de escrever um script de backup, é necessário confirmar onde os dados realmente residem em disco — ferramentas de orquestração (como o EasyPanel) podem usar estratégias de persistência (bind mounts) diferentes do que se assume por padrão (volumes Docker nomeados).
- Um script de automação que depende de uma lista fixa de nomes (clientes, serviços, etc.) cria débito de manutenção silencioso — preferir descoberta dinâmica baseada em uma convenção (como a estrutura de pastas) sempre que possível.
- Processos executados via `sudo` rodam no contexto do usuário `root`, incluindo arquivos de configuração de ferramentas de terceiros — qualquer configuração feita como um usuário comum precisa ser referenciada explicitamente ao automatizar via root.
- Editores de texto interativos no terminal (`nano`) podem ser inconsistentes dependendo do cliente SSH utilizado; para arquivos criados ou editados via scripts/automação, comandos não interativos (`cat`, `tee`, `sed`) são mais confiáveis e reproduzíveis.
- "Backup não testado é considerado inexistente": a validação de restauração revelou que o processo funciona de fato, algo que a simples ausência de erros no script de backup não garante por si só.
- Provedores de VPS frequentemente já incluem alguma camada básica de backup por padrão — vale verificar o que já está incluso no plano antes de assumir que uma solução paga é necessária.

---

*Documentação registrada em 02/08/2026.*
