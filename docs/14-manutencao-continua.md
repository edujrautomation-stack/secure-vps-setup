# 14 - Rotina de Manutenção Contínua

## Objetivo

Estabelecer um processo recorrente de revisão da infraestrutura, garantindo que as medidas de segurança implementadas nas fases anteriores (acesso seguro, firewall, Fail2ban, isolamento por cliente) continuem funcionando corretamente ao longo do tempo, e não apenas no momento em que foram configuradas.

---

## Motivação

Diferente das fases anteriores, que consistem em configurações pontuais aplicadas uma única vez, a segurança de uma infraestrutura em produção é um processo contínuo, não um estado permanente. Isso ocorre porque:

- proteções ativas (como o Fail2ban) podem falhar silenciosamente sem gerar alerta visível;
- atualizações de segurança do sistema operacional e das aplicações são publicadas constantemente, e adiar sua aplicação aumenta a janela de exposição a vulnerabilidades conhecidas;
- a entrada de novos clientes na infraestrutura compartilhada é o momento de maior risco de erro humano quanto ao isolamento de dados (Fase 9);
- credenciais e segundo fator de autenticação (2FA) podem ser desativados acidentalmente, ou nunca configurados corretamente em um serviço adicionado posteriormente.

A ausência de uma rotina formal de revisão é uma causa comum de degradação de segurança em infraestruturas que, na origem, foram configuradas corretamente.

---

## Tecnologias utilizadas

- Fail2ban (consulta de status)
- systemd / journalctl (consulta de logs do SSH)
- APT (verificação de atualizações do sistema)
- EasyPanel (verificação de versão via painel web)
- Bitwarden (geração e armazenamento de credenciais)

---

## Passo a passo

### 1. Revisão de logs do Fail2ban e do SSH

```bash
sudo fail2ban-client status sshd
```

**Objetivo:** confirmar que a jail do SSH continua ativa e monitorando corretamente, e obter uma visão geral de quantos IPs foram banidos desde a última reinicialização do serviço.

```bash
sudo journalctl -u ssh --since "30 days ago" | grep "Failed password"
```

**Objetivo:** revisar o volume de tentativas de login falhas no último mês. Um volume muito acima do padrão histórico pode indicar uma tentativa de ataque mais direcionada, diferente do tráfego constante de bots genéricos que qualquer servidor exposto recebe.

**Periodicidade recomendada:** mensal.

---

### 2. Verificação de atualizações — Docker e EasyPanel

```bash
sudo apt update && sudo apt list --upgradable
```

**Objetivo:** identificar atualizações pendentes do sistema operacional, incluindo pacotes de segurança e o próprio Docker (quando instalado via `apt`).

Para o EasyPanel, a verificação de versão é feita diretamente na interface web do painel, que normalmente sinaliza quando uma versão mais recente está disponível.

**Periodicidade recomendada:** mensal, com atenção especial a atualizações marcadas como críticas ou de segurança, que devem ser aplicadas o quanto antes, fora do ciclo mensal se necessário.

---

### 3. Revisão de 2FA e senhas a cada novo cliente onboardado

Diferente dos passos anteriores, esta etapa não depende de comandos no terminal, e sim de um processo de checagem manual no momento de onboarding de cada novo cliente:

- gerar uma senha ou API key exclusiva para o cliente, sempre através do gerador do Bitwarden — nunca reaproveitando credenciais de outro cliente ou serviço;
- confirmar que o 2FA das contas de infraestrutura (Hostinger, Cloudflare, EasyPanel) continua ativo, já que a adição de um novo cliente é um bom gatilho de checagem periódica, mesmo sem relação direta de causa e efeito.

**Objetivo:** garantir que o crescimento da operação não introduza credenciais fracas ou reaproveitadas, e que a superfície de proteção das contas administrativas não se degrade ao longo do tempo.

**Periodicidade recomendada:** a cada novo cliente onboardado.

---

### 4. Revisão de isolamento por cliente (Fase 9)

A cada novo cliente adicionado à infraestrutura, confirma-se que:

- existe um banco de dados PostgreSQL exclusivo para o cliente, nunca um schema compartilhado com outros clientes;
- existe uma instância própria de n8n e, quando aplicável, de Evolution API, exclusiva do cliente;
- os serviços criados no EasyPanel seguem uma nomenclatura clara e identificável (ex: `clienteX-postgres`, `clienteX-n8n`), facilitando auditoria visual rápida do ambiente.

**Objetivo:** evitar que um erro de configuração durante o onboarding resulte em vazamento de dados entre clientes diferentes hospedados na mesma infraestrutura compartilhada.

**Periodicidade recomendada:** a cada novo cliente onboardado.

---

## Como validar a configuração

Diferente das fases anteriores, esta fase não possui um estado final "concluído", mas sim indicadores de que a rotina está sendo seguida corretamente:

- ✅ Existe um registro (checklist ou planilha) das últimas revisões mensais realizadas
- ✅ Nenhuma atualização de segurança crítica está pendente há mais de 30 dias
- ✅ O volume de tentativas de login falhas no SSH está dentro do padrão histórico observado
- ✅ Todo cliente ativo na infraestrutura possui banco de dados, instância de n8n e (quando aplicável) instância de Evolution API exclusivos
- ✅ O 2FA das contas de infraestrutura (Hostinger, Cloudflare, EasyPanel) está ativo no momento da revisão

Um checklist mensal em formato PDF foi criado como ferramenta de apoio a essa rotina, com espaço para registrar o mês de referência, o responsável pela revisão e observações/pendências identificadas.

---

## Problemas encontrados e soluções

Esta fase, por sua natureza recorrente, não registra problemas técnicos de implementação como as anteriores — o principal risco identificado não é técnico, mas processual:

**1. Ausência de gatilho natural para lembrar da revisão.**

**Causa raiz:** ao contrário de uma configuração pontual, uma rotina mensal não possui nenhum evento do sistema que force sua execução — depende inteiramente de disciplina manual.

**Solução adotada:** criação de um checklist físico/digital (PDF) como lembrete estruturado, a ser preenchido no início de cada mês, associando a rotina a uma data fixa e reduzindo a chance de ser esquecida.

---

## Lições aprendidas

- Segurança de infraestrutura é um processo contínuo, não um estado alcançado e mantido automaticamente após a configuração inicial.
- Proteções automatizadas (como o Fail2ban) precisam de revisão periódica de seus próprios logs — a ausência de alertas não é, por si só, garantia de que estão funcionando como esperado.
- O momento de onboarding de um novo cliente é o ponto de maior risco de erro humano em uma infraestrutura multi-cliente, e por isso justifica uma checagem dedicada de isolamento e credenciais, além da rotina mensal padrão.
- Rotinas sem gatilho automático tendem a ser esquecidas; associar a revisão a um artefato concreto (checklist, planilha, lembrete de calendário) aumenta a probabilidade de execução consistente.
- Uma infraestrutura corretamente configurada na origem pode se degradar silenciosamente ao longo de meses sem uma rotina formal de revisão — a Fase 14 existe justamente para fechar esse ciclo.
