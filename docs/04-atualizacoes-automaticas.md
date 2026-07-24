# 04 - Atualizações Automáticas

## Objetivo

Configurar a aplicação automática de patches de segurança na VPS, eliminando a dependência de atualização manual e reduzindo o tempo de exposição a vulnerabilidades conhecidas.

---

# Motivação

Patches de segurança para o sistema operacional e pacotes instalados são publicados com frequência. Se a atualização depende de intervenção manual, existe uma janela de tempo em que o servidor permanece vulnerável a falhas já corrigidas publicamente.

Automatizar a aplicação de patches de segurança reduz essa janela de exposição sem exigir acompanhamento constante do administrador.

---

# Tecnologias utilizadas

- Ubuntu Server
- unattended-upgrades
- apt

---

# Passo a passo

## 1. Instalação do pacote

Verificado se o pacote `unattended-upgrades` já estava presente no sistema.

```bash
sudo apt install unattended-upgrades -y
```

### Objetivo

Garantir a presença do pacote responsável por gerenciar atualizações automáticas.

---

## 2. Ativação do serviço

Executada a configuração interativa do pacote, confirmando a ativação do download e instalação automática de atualizações estáveis.

```bash
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

### Objetivo

Registrar formalmente no sistema que as atualizações automáticas devem ser aplicadas.

---

## 3. Validação da ativação

```bash
cat /etc/apt/apt.conf.d/20auto-upgrades
```

Resultado confirmado:

```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

### Objetivo

Confirmar que a atualização automática de listas de pacotes e a instalação automática estão habilitadas.

---

## 4. Restrição às origens de segurança

Por padrão, o arquivo de configuração permite tanto atualizações gerais quanto atualizações de segurança. Para reduzir o risco de mudanças de comportamento não supervisionadas em produção, a origem de atualizações gerais foi desabilitada, mantendo apenas a origem de segurança.

Backup do arquivo original:

```bash
sudo cp /etc/apt/apt.conf.d/50unattended-upgrades /etc/apt/apt.conf.d/50unattended-upgrades.bak
```

Comentário da linha de atualizações gerais:

```bash
sudo sed -i 's|"${distro_id}:${distro_codename}";|// &|' /etc/apt/apt.conf.d/50unattended-upgrades
```

Validação:

```bash
cat /etc/apt/apt.conf.d/50unattended-upgrades | grep -A 5 "Allowed-Origins"
```

Resultado confirmado:

```
Unattended-Upgrade::Allowed-Origins {
        // "${distro_id}:${distro_codename}";
        "${distro_id}:${distro_codename}-security";
```

### Objetivo

Aplicar automaticamente apenas patches de segurança, mantendo atualizações gerais sob revisão manual.

---

## 5. Teste em modo simulação

```bash
sudo unattended-upgrade --dry-run --debug
```

Resultado:

```
No packages found that can be upgraded unattended and no pending auto-removals
```

### Objetivo

Validar o funcionamento do mecanismo sem aplicar alterações reais no sistema. A ausência de pacotes pendentes é esperada, já que o sistema havia sido atualizado manualmente em fase anterior.

---

# Como validar a configuração

As seguintes verificações foram realizadas:

✅ Pacote `unattended-upgrades` instalado
✅ Atualização automática ativada em `20auto-upgrades`
✅ Origem restrita a patches de segurança em `50unattended-upgrades`
✅ Backup do arquivo de configuração original preservado
✅ Simulação (`--dry-run`) executada sem erros

---

# Problemas encontrados e soluções

Nenhum imprevisto técnico ocorrido nesta fase. O pacote já estava presente no sistema, e a edição do arquivo de configuração foi feita via `sed`, evitando os problemas de navegação no `nano` observados em fases anteriores no terminal web.

---

# Lições aprendidas

- Atualização automática não deve ser confundida com atualização irrestrita: separar patches de segurança de atualizações gerais reduz o risco de mudanças inesperadas em produção.
- Sempre fazer backup de um arquivo de configuração antes de editá-lo, mesmo em alterações simples.
- O modo `--dry-run` permite validar a configuração de um mecanismo automático sem correr o risco de uma execução real indesejada.
- Editar arquivos de configuração via `sed` é mais confiável que `nano` em terminais web, onde atalhos de teclado podem ser interceptados pelo navegador.
