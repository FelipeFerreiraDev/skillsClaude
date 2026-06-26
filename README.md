# skills

Coleção de **skills**, **commands** e **agents** para o [Claude Code](https://claude.ai/code) — instruções reutilizáveis que estendem o comportamento do Claude com pipelines prontos para o fluxo de desenvolvimento.

## Skills disponíveis

### `dev-workflow`

Pipeline completo de desenvolvimento: levanta contexto, coleta tarefas, esclarece dúvidas, implementa as alterações, valida com o desenvolvedor e encadeia automaticamente o `git-commit-workflow` e o `git-pr-workflow`.

**Fluxo:** levantar contexto → coletar tarefas → implementar → revisar → commit → PR

**Gatilhos:** "quero implementar", "me ajuda a codar", "faz essas mudanças", "desenvolve isso pra mim", "preciso corrigir X e implementar Y", "implementa e já abre o PR", "resolve esses bugs"

---

### `git-commit-workflow`

Executa o pipeline completo de validação antes de commitar: build, testes, lint, formatação, revisão dos arquivos em stage, varredura de credenciais expostas, geração de mensagem semântica (Conventional Commits) e push.

**Fluxo:** detectar contexto → validar build/testes/lint → revisar stage → varrer credenciais → gerar mensagem → commit → push

**Gatilhos:** "commitar", "fazer commit", "push", "subir código", "enviar alterações", "git commit", "git push", "validar o código antes do commit", "checar qualidade do código", "preparar release"

---

### `git-pr-workflow`

Cria um Pull Request no GitHub com título semântico, descrição estruturada (o que foi feito, por que, como testar, impacto) e assinatura do autor. Suporta PT e EN.

**Fluxo:** verificar pré-requisitos → coletar contexto dos commits → gerar título e descrição → criar PR

**Gatilhos:** "abrir PR", "criar pull request", "criar PR", "open PR", "subir PR", "mandar PR"

---

### `create-readme`

Analisa o projeto automaticamente (ecossistema, entrypoints, scripts, variáveis de ambiente) e gera ou atualiza o `README.md` completo, confirmando com o desenvolvedor o que não pode ser inferido.

**Fluxo:** analisar estrutura → coletar metadados → explorar funcionalidades → verificar env vars → mapear scripts → perguntar lacunas → redigir → confirmar → salvar

**Gatilhos:** "cria um README", "gera documentação", "documenta o projeto", "faz o README", "atualiza o README", "melhora a documentação", "escreve o README para mim"

---

## Commands disponíveis

Commands são invocados explicitamente via slash command — o Claude não os dispara sozinho.

### `/multi-agents`

Instrui o Claude a despachar múltiplos subagentes em paralelo para resolver problemas independentes ao mesmo tempo. Use quando você tem N tarefas que não dependem uma da outra e quer que rodem concorrentemente.

**Quando usar:** múltiplos bugs em subsistemas distintos, partes independentes de uma feature grande, investigações que não compartilham estado.

**Quando não usar:** tarefas relacionadas (corrigir uma pode afetar a outra), quando você ainda não sabe o que está quebrado, quando os agentes editariam os mesmos arquivos.

---

## Agents disponíveis

Agents são personas especializadas invocadas pelo Claude como subagentes — geralmente via `/multi-agents` ou diretamente quando você precisa de um especialista pontual.

| Agent | Papel | Escreve código? |
|---|---|---|
| `unit-tester` | Testes unitários isolados com mocks | Só arquivos de teste |
| `integration-tester` | Testes de fluxo completo com infra real | Só arquivos de teste |
| `error-investigator` | Diagnóstico e causa raiz de falhas | Não — só relatório |
| `architect` | Design de software e plano de solução | Não — só plano |
| `database-architect` | Schema, índices, migrations | Só migrations |
| `db-security-auditor` | SQL injection, credenciais, permissões, `docs/db-audit.md` | Não — só auditoria |
| `documenter` | JSDoc, Swagger/OpenAPI, comentários inline | Só documentação |

---

## Quando usar o quê

### `dev-workflow` — uso padrão (80% dos casos)

Para qualquer tarefa coesa: um bug, uma feature, uma refatoração. O contexto flui de uma etapa para a próxima.

```
dev-workflow → implementa → git-commit-workflow → git-pr-workflow
```

### `/multi-agents` — quando há independência clara

Para quando você olha a lista de tarefas e percebe que elas não têm nada a ver uma com a outra. Ele não implementa — coordena agentes que implementam em paralelo.

```
/multi-agents → agente A + agente B + agente C (em paralelo)
```

### Combinação — tarefas grandes com partes independentes

```
architect           → desenha a solução
/multi-agents → implementa partes independentes em paralelo
unit-tester + integration-tester em paralelo → cobre tudo
git-commit-workflow + git-pr-workflow → fecha o ciclo
```

### Agents diretos — quando você precisa de um especialista pontual

```
error-investigator  → antes de tentar corrigir um bug obscuro
architect           → antes de começar uma feature complexa
documenter          → após implementar uma API
```

---

## Estrutura

```
skills/<nome>/SKILL.md    — pipeline com etapas numeradas
commands/<nome>.md        — slash commands invocados explicitamente
agents/<nome>/AGENT.md    — personas especializadas
```

## Instalação

### Global com symlink (recomendado)

Clone o repositório em qualquer lugar e aponte `~/.claude/skills/` para ele via symlink. Assim, um `git pull` já atualiza as skills sem nenhuma etapa extra.

```bash
git clone https://github.com/FelipeFerreiraDev/skillsClaude.git ~/dev/skills

# Criar symlinks para cada subdiretório dentro de ~/.claude/
ln -s ~/dev/skills/skills   ~/.claude/skills
ln -s ~/dev/skills/commands ~/.claude/commands
ln -s ~/dev/skills/agents   ~/.claude/agents
```

> Se algum dos diretórios já existir em `~/.claude/`, remova-o antes com `rmdir ~/.claude/<nome>` (só funciona se estiver vazio).

### Por projeto

Copie as skills para dentro do projeto (ficam disponíveis apenas nele):

```bash
cp -r skills/* /seu-projeto/.claude/skills/
```

---

As skills são invocadas automaticamente pelo Claude quando a intenção do usuário corresponde aos gatilhos descritos. Commands e agents são invocados explicitamente.

## Adicionando uma nova skill

1. Crie `skills/<nome>/SKILL.md` com frontmatter YAML:

```markdown
---
name: <slug>
description: >
  Frases de gatilho e descrição da intenção aqui.
---

## ETAPA 1 — ...
## ETAPA 2 — ...
```

2. Para expor como slash command direto, crie `commands/<nome>.md`:

```markdown
Execute a skill `/nome-da-skill` agora.
```

3. Consulte o [CLAUDE.md](CLAUDE.md) para convenções de formatação.
