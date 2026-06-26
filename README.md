# skills

Coleção de **skills** para o [Claude Code](https://claude.ai/code) — instruções reutilizáveis que estendem o comportamento do Claude com pipelines prontos para o fluxo de desenvolvimento.

## Skills disponíveis

### `dev-workflow`

Pipeline completo de desenvolvimento: levanta contexto, coleta tarefas, esclarece dúvidas, implementa as alterações, valida com o desenvolvedor e encadeia automaticamente o `git-commit-workflow` e o `git-pr-workflow`.

**Gatilhos:** "quero implementar", "me ajuda a codar", "faz essas mudanças", "preciso corrigir X e implementar Y", "implementa e já abre o PR", "resolve esses bugs"

---

### `git-commit-workflow`

Executa o pipeline completo de validação antes de commitar: build, testes, lint, formatação, revisão dos arquivos em stage, varredura de credenciais expostas, geração de mensagem semântica (Conventional Commits) e push.

**Gatilhos:** "commitar", "fazer commit", "push", "subir código", "git commit", "validar o código antes do commit"

---

### `git-pr-workflow`

Cria um Pull Request no GitHub com título semântico, descrição estruturada (o que foi feito, por que, como testar, impacto) e assinatura do autor. Suporta PT e EN.

**Gatilhos:** "abrir PR", "criar pull request", "subir PR", "mandar PR"

---

### `create-readme`

Analisa o projeto automaticamente (ecossistema, entrypoints, scripts, variáveis de ambiente) e gera um `README.md` completo, confirmando com o desenvolvedor o que não pode ser inferido.

**Gatilhos:** "cria um README", "gera documentação", "documenta o projeto", "atualiza o README"

---

## Estrutura

```
skills/<nome>/SKILL.md    — pipeline com etapas numeradas
commands/<nome>.md        — aliases de slash commands (delegam para uma skill)
agents/<nome>/            — personas especializadas
```

## Instalação

Clone este repositório dentro do diretório de skills do Claude Code:

```bash
# Opção 1 — projeto específico (copia as skills para .claude/skills/)
cp -r skills/* /seu-projeto/.claude/skills/

# Opção 2 — global (disponível em todos os projetos)
cp -r skills/* ~/.claude/skills/
```

As skills são invocadas automaticamente pelo Claude quando a intenção do usuário corresponde aos gatilhos descritos, ou manualmente via slash command (`/dev-workflow`, `/git-commit-workflow`, etc.).

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
