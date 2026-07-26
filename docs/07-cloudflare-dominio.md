# 07 - Cloudflare + Domínio

## Objetivo

Migrar o gerenciamento de DNS do domínio para a Cloudflare, ocultando o IP real da origem, forçando tráfego criptografado ponta a ponta e ativando proteções básicas contra bots e tráfego malicioso.

---

# Motivação

Acessar os serviços da VPS diretamente pelo IP expõe a origem a ataques diretos e não oferece certificado HTTPS válido. Colocar a Cloudflare como camada intermediária entre a internet e o servidor de origem permite:

- Ocultar o IP real da VPS
- Emitir e renovar certificados SSL automaticamente
- Filtrar tráfego malicioso e bots antes que cheguem à origem
- Ter uma camada gratuita de proteção contra DDoS

---

# Tecnologias utilizadas

- Cloudflare (plano Free)
- DNS / Nameservers (Hostinger, como registrador do domínio)
- Domínio: nexflowdx.cloud

---

# Passo a passo

## 1. Adição do domínio na Cloudflare

O domínio foi adicionado à Cloudflare através da opção "Add a domain", selecionando o plano Free. A ferramenta escaneou automaticamente os registros DNS existentes.

### Objetivo

Iniciar o processo de migração do gerenciamento de DNS para a Cloudflare.

---

## 2. Ajuste do registro DNS principal

O registro `A` do domínio raiz, originalmente apontando para um IP de parking da Hostinger, foi corrigido para apontar ao IP real da VPS, mantendo o proxy (nuvem laranja) ativado.

```
Tipo: A
Nome: nexflowdx.cloud
Conteúdo: <IP da VPS>
Proxy status: Proxied
TTL: Auto
```

### Objetivo

Garantir que as requisições ao domínio cheguem à VPS correta, passando pela rede da Cloudflare (que oculta o IP de origem).

---

## 3. Migração dos nameservers

No painel do registrador (Hostinger), os nameservers padrão foram substituídos pelos nameservers atribuídos pela Cloudflare.

Nameservers removidos:
```
athena.dns-parking.com
apollo.dns-parking.com
```

Nameservers adicionados:
```
miles.ns.cloudflare.com
sofia.ns.cloudflare.com
```

### Objetivo

Transferir a autoridade de DNS do domínio para a Cloudflare, permitindo que suas configurações (proxy, SSL, regras de segurança) tenham efeito.

Após a alteração, o domínio ficou em estado pendente de propagação até a Cloudflare confirmar a mudança (confirmação recebida por e-mail).

---

## 4. Configuração do SSL/TLS em modo Full (strict)

Em **SSL/TLS → Overview**, o modo de criptografia foi alterado de **Full** para **Full (strict)**.

### Objetivo

Criptografar a conexão entre a Cloudflare e o servidor de origem, validando também a autenticidade do certificado instalado na origem — evitando que a conexão aceite certificados autoassinados ou não confiáveis.

---

## 5. Ativação de Always Use HTTPS

Em **SSL/TLS → Edge Certificates**, a opção **Always Use HTTPS** foi ativada.

### Objetivo

Redirecionar automaticamente qualquer requisição feita via `http://` para `https://`, eliminando a possibilidade de tráfego não criptografado.

---

## 6. Ativação do Bot Fight Mode

Em **Security → Bots**, o **Bot Fight Mode** foi ativado.

### Objetivo

Identificar e desafiar automaticamente tráfego proveniente de bots automatizados e scanners maliciosos antes que atinjam a origem, sem impacto para visitantes legítimos.

---

# Item não implementado (deliberadamente)

## Regra de firewall básica (bloqueio geográfico/IP)

Esse item do roadmap foi avaliado e **não implementado nesta fase**, por decisão consciente:

O modelo de negócio atende clientes internacionais distribuídos em diferentes países. Um bloqueio geográfico amplo correria o risco de barrar o acesso de clientes legítimos tentando acessar seus próprios serviços (n8n, EasyPanel) a partir de regiões diversas. Nesse estágio, a proteção contra tráfego automatizado já é coberta pelo Bot Fight Mode, que atua sem depender de origem geográfica.

A criação de regras de bloqueio por IP específico fica reservada para casos pontuais — caso um ataque direcionado seja identificado a partir de um IP ou região específica no futuro, uma regra pontual poderá ser criada a qualquer momento, sem necessidade de planejamento prévio nesta fase.

---

# Como validar a configuração

As seguintes verificações foram realizadas:

✅ Domínio ativo na Cloudflare (confirmação por e-mail após propagação dos nameservers)
✅ Registro A apontando para o IP da VPS, com proxy ativado
✅ SSL/TLS em modo Full (strict) confirmado na tela de overview
✅ Always Use HTTPS ativado
✅ Bot Fight Mode ativado
✅ Certificado Universal SSL ativo, cobrindo `*.nexflowdx.cloud` e o domínio raiz

---

# Problemas encontrados e soluções

Um aviso de certificado ("This hostname is not covered by a certificate") apareceu durante o processo de ativação — comportamento esperado, já que o certificado só é emitido após a confirmação de que a Cloudflare controla o DNS do domínio, o que só ocorre depois da propagação dos nameservers. O aviso desapareceu automaticamente após a propagação ser concluída.

---

# Lições aprendidas

- A migração de nameservers é uma alteração de baixo risco de indisponibilidade, mas depende de propagação, que pode levar de minutos a horas.
- Avisos de certificado durante a configuração inicial não indicam necessariamente um erro — muitas vezes refletem apenas uma etapa anterior (propagação de DNS) ainda não concluída.
- Nem todo item de um roadmap de segurança genérico se aplica ao contexto real do negócio: bloqueio geográfico, por exemplo, pode ser contraproducente para uma operação com clientes internacionais distribuídos.
- O modo Full (strict) exige um certificado válido na origem — importante ter isso em mente antes de ativá-lo em um ambiente que ainda não tenha certificado configurado nos serviços internos.

---

*Documentação registrada em 26/07/2026.*
