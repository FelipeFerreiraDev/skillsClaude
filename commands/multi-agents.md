# Despachando Agentes em Paralelo

## Visão Geral

Você delega tarefas a agentes especializados com contexto isolado. Os agentes
**nunca herdam o contexto da sua sessão** — você monta exatamente o que eles
precisam. Isso preserva seu contexto para coordenação e permite execução
concorrente.

**Princípio central:** Despache um agente por domínio independente, todos na
mesma resposta. Múltiplas chamadas numa resposta = paralelo. Uma por resposta =
sequencial.

___

## Catálogo de agentes disponíveis

| Agente | Especialidade | Escreve código? |
|---|---|---|
| `architect` | Design de módulos, contratos, plano em `docs/architecture/` | Não (só docs) |
| `database-architect` | Schema, migrations, índices, estratégia de query | Migrations |
| `db-security-auditor` | SQL injection, credenciais expostas, permissões, `docs/db-audit.md` | Não |
| `documenter` | JSDoc, Swagger/OpenAPI, docstrings, comentários inline | Só docs |
| `error-investigator` | Diagnóstico de falhas, relatório em `docs/investigations/` | Não |
| `unit-tester` | Testes unitários com mocks/stubs, código isolado | Arquivos de teste |
| `integration-tester` | Testes ponta a ponta com infraestrutura real | Arquivos de teste |
| `node-backend` | Código de produção Node.js/TS — entidades, use cases, adapters, controllers (Express/NestJS + Prisma, Hexagonal/Clean Architecture) | Sim |
| `react-frontend` | Código de produção React/Vite/TS — componentes, hooks, serviços de API, rotas, CSS Modules | Sim |

**Agentes somente leitura** (`error-investigator`, `db-security-auditor`):
seguros para rodar em paralelo com qualquer outro — não escrevem nada além de relatórios
em `docs/`.

**Agentes de escrita com destinos distintos — paralelo seguro entre si:**
- `documenter` → escreve em arquivos de produção (adiciona JSDoc/Swagger/comentários)
- `unit-tester` → escreve arquivos de teste unitário (`*.spec.ts`)
- `integration-tester` → escreve arquivos de teste de integração (`*.e2e-spec.ts`)

Os três escrevem em arquivos diferentes por natureza, portanto **podem sempre rodar
em paralelo entre si e com `error-investigator` / `db-security-auditor`**.

**Agentes de escrita que exigem verificação antes de paralelizar** (`architect`,
`database-architect`, `node-backend`, `react-frontend`): verifique se escrevem
em arquivos diferentes antes de rodar em paralelo.

___

## ETAPA 1 — Identificar os domínios independentes

Agrupe os problemas pelo que está quebrado ou pelo que precisa ser feito. Cada
domínio deve poder ser tratado sem depender dos outros.

**Use quando:**
- 3+ problemas com causas raiz diferentes
- Múltiplos subsistemas independentes
- Tarefas sem estado compartilhado (ex: documentar módulo A enquanto testa módulo B)

**Não use quando:**
- As tarefas têm dependência de ordem (arquitetura deve vir antes da implementação)
- Agentes precisariam editar o mesmo arquivo
- A investigação ainda é exploratória — você ainda não sabe o que está quebrado

___

## ETAPA 2 — Criar tarefas focadas para cada agente

Cada agente precisa de contexto completo no prompt — ele não herda nada da sessão.

**Inclua sempre:**
- Escopo exato (arquivo, módulo, subsistema)
- Objetivo claro e resultado esperado
- Restrições (o que não deve ser tocado)
- Contexto mínimo necessário (stack, padrões relevantes)

**Modelo de prompt:**

```markdown
Investigue as 3 falhas em `src/auth/token.service.ts`:

1. "should expire token after 1h" — retorna null em vez de lançar erro
2. "should reject tampered payload" — passa quando deveria falhar
3. "should refresh token within grace period" — race condition suspeita

Contexto: projeto NestJS, JWT via `@nestjs/jwt`, sem Redis ainda.

Sua tarefa:
1. Leia o arquivo de testes e o código de produção
2. Identifique a causa raiz de cada falha
3. Salve o relatório em docs/investigations/

NÃO altere nenhum código. Retorne: causa raiz por falha e recomendação de correção.
```

**Erros comuns:**

| Errado | Certo |
|---|---|
| "Corrija todos os testes" | "Corrija `token.service.test.ts`" |
| "Há uma race condition" | Cole a mensagem de erro exata |
| Sem restrições | "NÃO altere código de produção" |
| "Resolva isso" | "Retorne resumo da causa raiz e mudanças" |

___

## ETAPA 3 — Despachar em paralelo

Emita todos os despachos na **mesma resposta**:

```
Agente 1 (error-investigator) → Diagnosticar falhas em token.service.ts
Agente 2 (unit-tester)        → Escrever testes para payment.service.ts
Agente 3 (documenter)         → Documentar rotas de auth/
# Todos rodam concorrentemente — domínios independentes, sem conflito de arquivo.
```

___

## Combinações comuns

**Diagnóstico + cobertura em paralelo**
```
error-investigator → identifica causa raiz do módulo A
unit-tester        → escreve testes para o módulo B (não relacionado)
```

**Planejamento completo de feature nova**
```
architect          → propõe estrutura de módulos e contratos
database-architect → propõe schema e migrations
# Rodam em paralelo; o plano de arquitetura e o plano de BD são independentes.
```

**Auditoria paralela**
```
db-security-auditor → audita queries e credenciais
documenter          → documenta endpoints da API
# Ambos são somente leitura — zero risco de conflito.
```

**Cobertura de teste completa**
```
unit-tester        → testes unitários de `user.service.ts`
integration-tester → testes de integração do fluxo de cadastro
# Escrevem arquivos diferentes — sem conflito.
```

**Cobertura + documentação em paralelo (combinação máxima segura)**
```
unit-tester        → testes unitários de `payment.service.ts`
integration-tester → testes de integração do fluxo de checkout
documenter         → documenta endpoints de `order.controller.ts` com Swagger
# Três agentes simultâneos: cada um escreve em arquivos de natureza diferente.
# unit/integration → arquivos de teste; documenter → comentários no código de produção.
```

**Feature fullstack em paralelo**
```
node-backend   → implementa use case + controller + repository da feature
react-frontend → implementa página + componentes + service de API
# Rodam em paralelo; escrevem em diretórios distintos (src/backend vs src/frontend ou monorepo).
# CUIDADO: alinhe o contrato da API (tipos de request/response) antes de despachar.
```

**Implementação + cobertura simultânea**
```
node-backend    → implementa o módulo de pagamento
unit-tester         → escreve testes unitários de um módulo já existente (não relacionado)
# O unit-tester não toca no código novo — sem conflito.
```

**Implementação + documentação**
```
node-backend   → implementa endpoints de autenticação
documenter           → documenta endpoints de produto já existentes com Swagger/OpenAPI
# O documenter só adiciona JSDoc — não conflita com o código novo.
```

___

## ETAPA 4 — Revisar e integrar

Quando os agentes retornarem:

1. **Leia cada resumo** — entenda o que mudou ou foi encontrado
2. **Verifique conflitos** — algum agente editou o mesmo arquivo que outro?
3. **Valide a consistência** — os planos do `architect` e `database-architect` são coerentes?
4. **Rode a suíte completa** — verifique que tudo funciona junto
5. **Faça spot check** — agentes podem cometer erros sistemáticos; revise os arquivos críticos
