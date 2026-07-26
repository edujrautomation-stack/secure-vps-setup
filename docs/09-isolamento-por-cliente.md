# 09 - Isolamento por Cliente

## Objetivo

Definir a convenção de nomenclatura e a estratégia de isolamento lógico entre clientes que vão compartilhar a mesma VPS, antes da criação de qualquer instância real de banco de dados ou aplicação.

---

# Motivação

Uma VPS compartilhada atendendo múltiplos clientes precisa garantir que dados, credenciais e workflows de um cliente nunca fiquem acessíveis a outro. Diferente de um erro técnico isolado, um vazamento cruzado entre clientes é o tipo de incidente que compromete a confiança e a reputação de um freelancer que atende múltiplas empresas na mesma infraestrutura.

Como ainda não havia nenhum cliente ativo neste momento, esta fase teve como objetivo estabelecer o padrão antecipadamente, para que a criação de instâncias reais (Fases 10 e 11) siga uma convenção já validada, em vez de decisões tomadas sob pressão no momento do primeiro onboarding.

---

# Estratégia de isolamento definida

## 1. Isolamento por instância, não por schema compartilhado

- **Postgres:** cada cliente recebe seu próprio container/instância de banco de dados, não um schema dentro de uma instância compartilhada. Isso contém o impacto de um bug de query ou uma permissão mal configurada a um único cliente.
- **n8n:** cada cliente recebe sua própria instância, não um workspace dentro de uma instância única — evitando que credenciais de workflows de um cliente fiquem visíveis para outro.
- **Evolution API:** cada cliente recebe sua própria instância, com API key própria, garantindo que sessões de WhatsApp e dados de conversa fiquem isolados.

## 2. Convenção de nomenclatura

Padrão definido, usando um identificador curto por cliente:

```
<cliente>-postgres
<cliente>-n8n
<cliente>-evolution
```

Subdomínios seguem o mesmo padrão:

```
<cliente>-n8n.nexflowdx.cloud
```

### Objetivo da convenção

Nomes curtos e padronizados permitem auditoria visual rápida na lista de serviços do EasyPanel, reduzindo o risco de confundir qual serviço pertence a qual cliente.

---

# Clientes de exemplo definidos para treino

Para validar o padrão antes da chegada de clientes em produção, foram definidos três clientes de exemplo — um real e dois fictícios:

| Cliente | Identificador | Tipo |
|---|---|---|
| Revendedora Ultragas de Pirituba | `ultragas` | Real (primeiro cliente) |
| NexflowDX | `nexflow` | Fictício (empresa própria) |
| Massas duDú | `dudu` | Fictício |

Esses identificadores serão usados como exemplo prático na criação das instâncias nas Fases 10 e 11.

---

# Escopo definido: Postgres puro, sem Supabase

Durante esta fase, foi avaliada a possibilidade de usar o Supabase como camada de banco de dados para os clientes. A decisão foi **não utilizar o Supabase para essa finalidade**:

- n8n e Evolution API precisam apenas de uma conexão Postgres padrão — não utilizam autenticação, API REST automática, realtime ou storage, que são os diferenciais do Supabase.
- Rodar o Supabase self-hosted implica em containers adicionais (Auth, Realtime, Storage, API Gateway, Studio) consumindo recursos da VPS sem benefício funcional para este caso de uso.
- Mais serviços rodando também aumentam a superfície de manutenção e atualização sem ganho correspondente.

A instância de Postgres por cliente, definida na estratégia de isolamento acima, utiliza a imagem oficial e leve do Postgres — sem relação com o Supabase.

O Supabase será avaliado e instalado separadamente, fora do escopo deste roadmap, para atender a um projeto futuro que efetivamente precisa de suas funcionalidades (autenticação de usuários, API automática, storage).

---

# Como validar a configuração

Como esta fase não envolveu execução de comandos na VPS, a validação se deu pela revisão da convenção definida:

✅ Estratégia de isolamento (instância própria por serviço, por cliente) documentada
✅ Convenção de nomenclatura definida e testável
✅ Três clientes de exemplo (um real, dois fictícios) definidos para uso nas próximas fases
✅ Decisão sobre Postgres vs. Supabase avaliada e justificada

---

# Problemas encontrados e soluções

Não houve problemas técnicos nesta fase, por se tratar de uma etapa de planejamento e definição de padrão, sem execução de comandos.

---

# Lições aprendidas

- Nem toda fase de um roadmap de infraestrutura envolve comandos — definir convenções e processos antes de criar recursos reais evita retrabalho e inconsistência quando o volume de clientes crescer.
- Isolamento lógico por instância separada é uma defesa mais robusta contra vazamento cruzado de dados do que segmentação por schema ou workspace dentro de uma instância compartilhada.
- Ferramentas com funcionalidades extras (como o Supabase) não devem ser adotadas por padrão apenas por estarem disponíveis — o critério é se as funcionalidades adicionais realmente serão usadas no caso de uso em questão.
- Usar exemplos fictícios ao lado de um caso real permite validar um processo (nomenclatura, criação de instâncias) sem risco de erro em dados de produção.

---

*Documentação registrada em 26/07/2026.*
