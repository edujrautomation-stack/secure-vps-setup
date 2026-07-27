# 10 - Bancos de Dados (Postgres + Redis)

## Objetivo

Provisionar as instâncias de banco de dados (Postgres) e de suporte a filas (Redis) para cada cliente de exemplo, aplicando a estratégia de isolamento definida na Fase 9, sem exposição pública de portas.

---

# Motivação

Com a convenção de isolamento por cliente já definida na fase anterior, esta etapa coloca o padrão em prática: cada cliente recebe seu próprio Postgres e seu próprio Redis, isolados em projetos separados no EasyPanel, garantindo que uma falha ou vazamento em um serviço não afete os demais clientes.

---

# Tecnologias utilizadas

- EasyPanel (Projects, isolamento de rede Docker por projeto)
- PostgreSQL 17
- Redis
- Bitwarden (armazenamento de credenciais)

---

# Passo a passo

## 1. Criação dos projetos por cliente

Foram criados três projetos no EasyPanel, um por cliente, cada um com sua própria rede interna Docker isolada:

```
ultragas   (cliente real — Revendedora Ultragas de Pirituba)
nexflow    (cliente fictício — NexflowDX)
dudu       (cliente fictício — Massas duDú)
```

### Objetivo

Cada projeto no EasyPanel corresponde a uma rede Docker isolada — a separação por projeto reforça o isolamento lógico definido na Fase 9, não apenas organizando visualmente os serviços, mas isolando a comunicação de rede entre eles.

---

## 2. Criação do Postgres por cliente

Dentro de cada projeto, foi criado um serviço Postgres com o template padrão do EasyPanel:

```
ultragas-postgres   (banco: ultragas)
nexflow-postgres    (banco: nexflow)
dudu-postgres        (banco: dudu)
```

Configuração utilizada em cada um:

- Usuário e senha gerados automaticamente pelo EasyPanel (não preenchidos manualmente)
- Imagem Docker padrão (versão oficial mais recente — Postgres 17)

### Objetivo

Cada cliente recebe uma instância de banco de dados completamente separada — não um schema dentro de um banco compartilhado — eliminando o risco de uma query ou permissão mal configurada expor dados entre clientes.

---

## 3. Criação do Redis por cliente

Dentro de cada projeto, foi criado também um serviço Redis:

```
ultragas-redis
nexflow-redis
dudu-redis
```

### Objetivo

O Redis é utilizado internamente pelo n8n para gerenciamento de filas de execução de workflows. Manter uma instância própria por cliente evita que filas de processamento de clientes diferentes se misturem.

---

## 4. Validação de não exposição pública

Para cada serviço Redis criado, foi verificada a aba **"Expor"**, confirmando que o campo "Porta Exposta" permanecia em `0`.

### Objetivo

Garantir que os bancos de dados permaneçam acessíveis apenas pela rede interna do Docker, dentro do próprio projeto — nunca diretamente pela internet. Essa é uma verificação de segurança básica que deve ser repetida para qualquer serviço de banco de dados criado no futuro.

---

## 5. Armazenamento das credenciais

Para cada uma das seis instâncias criadas (3 Postgres + 3 Redis), as credenciais geradas automaticamente (usuário, senha, host interno, porta) foram copiadas da aba **"Credenciais"** de cada serviço e salvas no Bitwarden, na pasta **"Nexflow DX - Clientes"**.

### Objetivo

Evitar que credenciais de banco de dados fiquem registradas apenas na interface do EasyPanel ou em anotações informais — centralizar tudo no gerenciador de senhas, seguindo a prática já estabelecida desde a Fase 0.

---

# Decisão de escopo: Supabase fora deste roadmap

Durante o planejamento desta fase (ver Fase 9), foi avaliada e descartada a adoção do Supabase como camada de banco de dados para os clientes, por não agregar funcionalidade ao caso de uso do n8n/Evolution API. O Postgres utilizado aqui é a imagem oficial padrão, sem relação com o Supabase, que será avaliado separadamente para um projeto futuro distinto.

---

# Como validar a configuração

As seguintes verificações foram realizadas:

✅ Três projetos criados no EasyPanel, um por cliente
✅ Um Postgres criado por projeto, com banco de dados nomeado por cliente
✅ Um Redis criado por projeto
✅ Porta Exposta confirmada em `0` (sem exposição pública) em todos os serviços de banco de dados
✅ Logs do Postgres confirmando "database system is ready to accept connections" em todas as instâncias
✅ Credenciais das seis instâncias salvas no Bitwarden

---

# Problemas encontrados e soluções

Nenhum imprevisto técnico ocorrido nesta fase. O template padrão do EasyPanel já cria os serviços com as configurações de segurança corretas por padrão (sem exposição de porta), exigindo apenas verificação, não correção.

---

# Lições aprendidas

- No EasyPanel, um projeto não é apenas uma pasta organizacional — ele define uma rede Docker isolada, o que tem impacto direto na estratégia de segurança e isolamento entre clientes.
- Verificar a configuração de exposição de porta é uma checagem rápida e barata que deve ser feita para todo serviço de banco de dados criado, mesmo quando o padrão da ferramenta já vem seguro.
- Salvar credenciais no gerenciador de senhas imediatamente após a criação de cada serviço evita esquecimentos que se acumulam quando vários serviços são criados em sequência.
- Definir a convenção de nomenclatura e a estratégia de isolamento antes de criar qualquer recurso (Fase 9) tornou a execução desta fase mecânica e sem decisões pendentes no meio do processo.

---

*Documentação registrada em 26/07/2026.*
