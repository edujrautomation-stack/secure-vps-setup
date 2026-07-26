# 08 - HTTPS Automático

## Objetivo

Confirmar a emissão automática de certificado SSL válido (Let's Encrypt) na origem, via EasyPanel/Traefik, completando a segunda etapa de criptografia ponta a ponta exigida pelo modo SSL/TLS Full (strict) configurado na Cloudflare.

---

# Motivação

O modo Full (strict), ativado na Fase 7, criptografa e valida a conexão entre a Cloudflare e o servidor de origem. Para que isso funcione, a origem (VPS) precisa apresentar um certificado SSL válido, emitido por uma autoridade certificadora reconhecida — não um certificado autoassinado.

O EasyPanel utiliza o Traefik como proxy reverso interno, que é capaz de solicitar e renovar automaticamente certificados gratuitos da Let's Encrypt para qualquer domínio configurado em um serviço, sem intervenção manual contínua.

---

# Tecnologias utilizadas

- EasyPanel
- Traefik (proxy reverso interno do EasyPanel)
- Let's Encrypt
- Cloudflare DNS

---

# Passo a passo

## 1. Criação do registro DNS de teste

Foi criado um registro `A` para o subdomínio do painel administrativo, temporariamente sem proxy (DNS only), para permitir a validação direta do domínio pela Let's Encrypt.

```
Tipo: A
Nome: painel
Conteúdo: <IP da VPS>
Proxy status: DNS only (temporário)
TTL: Auto
```

### Objetivo

Permitir que a Let's Encrypt acesse o servidor de origem diretamente durante o desafio de validação (HTTP challenge), sem a camada de proxy da Cloudflare interceptando a requisição nesse momento inicial.

### Observação de segurança

Com o proxy desativado, o IP real do servidor fica temporariamente exposto — a própria Cloudflare exibe um aviso informativo sobre isso. O firewall (UFW) e o Fail2ban, configurados nas fases anteriores, continuam ativos durante esse intervalo, mitigando o risco da exposição temporária.

---

## 2. Configuração do domínio personalizado no EasyPanel

Em **Settings → Geral → Domínio do Painel**, o campo "Domínio Personalizado" foi preenchido com `painel.nexflowdx.cloud`, e a configuração foi salva.

### Objetivo

Instruir o EasyPanel a servir sua interface administrativa através do novo domínio, disparando a solicitação automática de certificado pelo Traefik.

---

## 3. Validação da emissão do certificado

O acesso via `https://painel.nexflowdx.cloud` foi testado diretamente no navegador, confirmando conexão segura sem avisos de certificado inválido.

### Objetivo

Confirmar que o Traefik solicitou e recebeu com sucesso um certificado válido da Let's Encrypt para o novo domínio.

---

## 4. Reativação do proxy da Cloudflare

Após a confirmação da emissão do certificado, o registro DNS `painel` foi atualizado de "DNS only" para "Proxied" novamente.

### Objetivo

Restaurar a proteção completa da Cloudflare (ocultação do IP de origem, mitigação de DDoS, filtragem de tráfego) agora que o certificado de origem já está validado e não depende mais de acesso direto ao servidor.

O acesso via `https://painel.nexflowdx.cloud` foi testado novamente após a reativação, confirmando funcionamento normal.

---

# Como validar a configuração

As seguintes verificações foram realizadas:

✅ Registro DNS do subdomínio criado e apontando para a VPS
✅ Domínio personalizado configurado no EasyPanel
✅ Certificado Let's Encrypt emitido automaticamente, sem intervenção manual além da configuração do domínio
✅ Acesso HTTPS validado antes e depois da reativação do proxy da Cloudflare

---

# Problemas encontrados e soluções

Nenhum imprevisto técnico ocorreu nesta fase. O processo de emissão automática de certificado funcionou conforme esperado após o ajuste temporário do proxy DNS.

---

# Lições aprendidas

- A emissão de certificados via Let's Encrypt em ambientes atrás de proxy (Cloudflare) pode exigir a desativação temporária do proxy durante a validação inicial (desafio HTTP), a depender da configuração do proxy reverso da origem.
- Expor o IP de origem temporariamente durante um processo de validação controlado é um risco aceitável quando outras camadas de proteção (firewall, Fail2ban) já estão ativas.
- Uma vez emitido, o certificado permanece válido e gerenciado automaticamente pelo Traefik, incluindo renovação — não é necessário repetir esse processo manualmente para o mesmo domínio.
- Validar o acesso HTTPS tanto antes quanto depois de reativar o proxy evita assumir que a configuração está correta sem confirmação real.

---

*Documentação registrada em 26/07/2026.*
