# 11 - n8n e Evolution API

## Objetivo

Provisionar instâncias de n8n para cada cliente, conectadas ao Postgres e Redis isolados criados na Fase 10, com autenticação própria e chave de criptografia única por instância. Validar o processo completo com uma instância de Evolution API, incluindo banco de dados dedicado e chave de autenticação própria, acessível sempre via domínio HTTPS.

---

# Motivação

Com o padrão de isolamento definido (Fase 9) e a infraestrutura de banco de dados pronta (Fase 10), esta fase conecta as aplicações de automação (n8n) e de mensageria (Evolution API) a essa infraestrutura, mantendo o mesmo princípio: cada cliente opera de forma isolada, sem compartilhar credenciais, chaves de criptografia ou bancos de dados com outros clientes.

O processo foi validado nos três clientes de exemplo definidos na Fase 9 — o real (Revendedora Ultragas de Pirituba) e dois fictícios (NexflowDX, Massas duDú) — permitindo treinar o fluxo completo, incluindo o diagnóstico de um problema real de configuração, antes de repeti-lo em produção com clientes futuros.

---

# Tecnologias utilizadas

- EasyPanel (Modelos/Templates)
- n8n
- Evolution API v2.3.7
- PostgreSQL (bancos criados via terminal `psql`)
- Redis
- Bitwarden

---

# Passo a passo — n8n

## 1. Criação da instância via template

Para cada cliente, foi criada uma instância de n8n a partir do template do EasyPanel, dentro do respectivo projeto:

```
ultragas-n8n   (projeto: ultragas)
nexflow-n8n    (projeto: nexflow)
dudu-n8n       (projeto: dudu)
```

### Objetivo

Isolar cada instância de n8n dentro da rede Docker do projeto do cliente correspondente, seguindo a convenção definida na Fase 9.

---

## 2. Reconexão ao banco de dados correto

O template de n8n do EasyPanel cria automaticamente um banco de dados Postgres próprio (ex: `ultragas-n8n-db`), separado dos bancos já criados manualmente na Fase 10. Para evitar duplicidade de infraestrutura, as variáveis de ambiente de cada instância foram ajustadas para apontar para o Postgres do cliente já existente:

```
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=<cliente>_<cliente>-postgres
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=<cliente>
DB_POSTGRESDB_USER=postgres
DB_POSTGRESDB_PASSWORD=<senha do <cliente>-postgres>
```

Após confirmar o funcionamento da instância com o banco correto, o banco duplicado gerado pelo template (`<cliente>-n8n-db`) foi excluído em cada projeto.

### Objetivo

Evitar dois bancos de dados Postgres redundantes por cliente, mantendo a infraestrutura consolidada na instância criada na Fase 10.

---

## 3. Ativação de autenticação própria

Em cada instância, foram adicionadas as variáveis:

```
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=<senha forte gerada por cliente>
```

### Objetivo

Impedir acesso não autenticado ao editor de workflows. Na prática, a versão do n8n utilizada substitui o fluxo de Basic Auth por uma tela de configuração de conta de proprietário (**"Set up owner account"**) no primeiro acesso — o resultado de segurança é equivalente (login obrigatório), ainda que o mecanismo de autenticação em si seja gerido pela própria aplicação.

Cada instância recebeu, adicionalmente, uma `N8N_ENCRYPTION_KEY` própria e única, gerada automaticamente pelo template — a chave que o n8n utiliza para embaralhar credenciais salvas nos workflows, reforçando o isolamento entre clientes mesmo a nível de dado criptografado.

---

# Passo a passo — Evolution API

A instância de Evolution API foi implementada e validada por completo no cliente `ultragas`, servindo como processo de referência a ser replicado para os demais clientes.

## 1. Criação da instância via template

```
ultragas-evolution   (projeto: ultragas)
```

## 2. Banco de dados dedicado dentro do Postgres existente

Diferente do n8n (que usa o banco `ultragas` já existente), foi criado um banco adicional exclusivo para a Evolution API dentro da mesma instância de Postgres do cliente, evitando conflito de tabelas entre as duas aplicações:

```sql
CREATE DATABASE ultragas_evolution;
```

Comando executado via terminal interativo (`bash`), acessado diretamente no container do serviço `ultragas-postgres`, confirmando antes a disponibilidade do `psql`:

```bash
psql --version
psql -U postgres -c "CREATE DATABASE ultragas_evolution;"
```

### Objetivo

Manter a Evolution API na mesma instância de banco do cliente (sem criar um Postgres redundante), mas em um banco de dados próprio, evitando qualquer sobreposição de schema com o banco usado pelo n8n.

## 3. Conexão com Postgres e Redis existentes

```
DATABASE_CONNECTION_URI=postgres://postgres:<senha>@ultragas_ultragas-postgres:5432/ultragas_evolution
CACHE_REDIS_URI=redis://default:<senha>@ultragas_ultragas-redis:6379
```

## 4. Chave de autenticação própria

```
AUTHENTICATION_API_KEY=<chave forte e única, gerada no Bitwarden>
```

### Objetivo

Cada cliente possui uma API key exclusiva para autenticar chamadas à sua instância de Evolution API, garantindo que integrações de um cliente não consigam acessar dados ou disparar mensagens em nome de outro.

---

# Problema encontrado e solução: caracteres especiais em senhas

Durante a configuração, duas tentativas de conexão falharam com o erro `password authentication failed for user "postgres"` (no n8n do cliente `nexflow`) e posteriormente `Error P1000: Authentication failed` (na Evolution API do cliente `ultragas`).

**Causa raiz identificada:** as senhas dos bancos de dados continham caracteres especiais (`#` e `&`) que, no formato de variáveis de ambiente e nas URLs de conexão utilizadas pelo EasyPanel, foram interpretados de forma diferente do esperado — o `#` sendo tratado como início de comentário (truncando o restante da senha), e o `&` interferindo na leitura da URL de conexão.

**Solução aplicada:** as senhas afetadas foram substituídas por versões geradas sem caracteres especiais problemáticos (apenas letras e números), atualizadas em todas as instâncias que referenciavam o banco correspondente (n8n e Evolution API do mesmo cliente), seguidas de nova implantação.

### Lição para próximos clientes

Ao gerar senhas de banco de dados no Bitwarden para uso neste ambiente, restringir a caracteres alfanuméricos, evitando símbolos como `#`, `&`, `$`, `;`, `@`, que têm significado especial em arquivos de configuração e URLs de conexão.

---

# Como validar a configuração

As seguintes verificações foram realizadas:

✅ n8n criado e funcional para os 3 clientes de exemplo (ultragas, nexflow, dudu), cada um conectado ao seu próprio Postgres
✅ Bancos de dados duplicados dos templates (`<cliente>-n8n-db`) identificados e removidos
✅ Autenticação de proprietário configurada em cada instância de n8n
✅ Evolution API validada de ponta a ponta no cliente `ultragas`: banco de dados dedicado criado, 57 migrações do Prisma aplicadas com sucesso, conexão Redis estabelecida (`redis ready`), servidor HTTP ativo na porta 8080
✅ Erro de autenticação por caractere especial em senha diagnosticado e corrigido, com atualização replicada em todas as instâncias afetadas

---

# Problemas encontrados e soluções

Além do problema de caracteres especiais em senhas (detalhado acima), houve um erro operacional durante a criação da primeira instância de n8n: o serviço foi criado dentro do projeto errado (`dudu`, em vez de `ultragas`), devido à seleção incorreta do projeto ativo no EasyPanel antes de acessar o menu de Modelos. O erro foi identificado pela análise da URL do painel e do projeto exibido no menu lateral, e corrigido excluindo os serviços criados no local errado e refazendo a criação com o projeto correto selecionado.

---

# Lições aprendidas

- Templates prontos de aplicações (n8n, Evolution API) frequentemente provisionam seus próprios bancos de dados por padrão — é necessário revisar e redirecionar essas conexões manualmente quando já existe uma infraestrutura de banco de dados definida previamente, para evitar duplicidade de recursos.
- Caracteres especiais em senhas, embora aumentem a entropia teórica, podem quebrar a interpretação de variáveis de ambiente e URLs de conexão em determinadas plataformas — vale restringir a geração de senhas de infraestrutura a caracteres alfanuméricos.
- Confirmar visualmente o projeto/contexto ativo (via URL ou menu) antes de criar um recurso é uma verificação simples que evita retrabalho, especialmente em plataformas com múltiplos projetos isolados.
- Validar um processo complexo (múltiplas variáveis de ambiente, múltiplos serviços dependentes) em um cliente de referência antes de replicá-lo em massa permite isolar e corrigir problemas uma única vez, em vez de depurá-los repetidamente.

---

*Documentação registrada em 01/08/2026.*
