# 00 - Preparação Inicial

## Objetivo

Proteger as contas principais utilizadas na infraestrutura antes de iniciar a configuração da VPS.

A primeira etapa de segurança consiste em habilitar autenticação em dois fatores (2FA), reduzindo o risco de acesso não autorizado às plataformas que irão gerenciar a infraestrutura.

---

# 1. Autenticação em dois fatores (2FA)

## Hostinger

### Objetivo

Adicionar uma camada extra de segurança à conta Hostinger, impedindo que um acesso indevido baseado apenas em senha comprometa a infraestrutura.

### Motivação

A conta Hostinger terá acesso ao gerenciamento da VPS. Caso essa conta seja comprometida, um invasor poderia alterar configurações, acessar recursos ou comprometer serviços hospedados.

Por isso, a autenticação em dois fatores foi ativada antes de iniciar qualquer configuração da VPS.

### Tecnologias utilizadas

- Conta Hostinger
- Autenticação em dois fatores (2FA)
- Aplicativo autenticador

### Passo a passo realizado

- Acessado o painel da Hostinger
- Acessado as configurações de segurança da conta
- Ativada a autenticação em dois fatores
- Vinculado o aplicativo autenticador
- Confirmada a ativação do segundo fator

### Como validar a configuração

A configuração foi validada realizando um novo login na conta Hostinger e verificando a solicitação do segundo fator de autenticação.

Status:

✅ Concluído

Data da configuração:

22/07/2026

---

## Cloudflare

### Objetivo

Proteger a conta Cloudflare, que será utilizada futuramente para gerenciamento de DNS, certificados SSL, proteção de domínio e publicação segura de serviços.

### Motivação

A Cloudflare será uma peça importante da infraestrutura. Um acesso não autorizado poderia permitir alterações de DNS, redirecionamentos indevidos ou exposição de serviços.

A ativação de múltiplos fatores de autenticação reduz o risco de comprometimento da conta.

### Tecnologias utilizadas

- Cloudflare
- Autenticação de dois fatores (2FA)
- Chave de segurança física/digital
- Aplicativo autenticador em dispositivo móvel

### Passo a passo realizado

- Acessado o painel da Cloudflare
- Acessado as configurações de segurança da conta
- Ativada autenticação de dois fatores por chave de segurança
- Ativada autenticação de dois fatores em dispositivo móvel
- Confirmadas as opções de proteção da conta

### Como validar a configuração

A configuração foi validada verificando no painel de segurança da Cloudflare que os seguintes métodos estão ativos:

✅ Autenticação de dois fatores por chave de segurança  
✅ Autenticação de dois fatores em dispositivo móvel

Status:

✅ Concluído

Data da configuração:

22/07/2026

---

# Lições aprendidas

- Contas de infraestrutura devem ser protegidas antes de criar serviços públicos.
- Utilizar mais de um método de autenticação aumenta a resiliência da conta.
- Chaves de segurança oferecem uma camada adicional de proteção contra ataques baseados em roubo de senha ou phishing.
