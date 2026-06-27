# CLAUDE.md

Este arquivo fornece orientações ao Claude Code (claude.ai/code) ao trabalhar neste repositório.

## O que é este repositório

Uma coleção de **skills**, **commands** e **agents** para o Claude Code — conjuntos de instruções reutilizáveis que estendem o comportamento do Claude. Nada aqui compila ou executa; todos os artefatos são arquivos Markdown.

## Estrutura

```
skills/<nome>/SKILL.md    — Skills: workflows multi-etapa disparados por intenção
commands/<nome>.md        — Commands: aliases diretos de slash commands
agents/<nome>/            — Agents: personas especializadas
docs/                     — Documentação do repositório (fluxos, guias de uso)
```

## Formato de skill

Toda skill vive em `skills/<nome>/SKILL.md` com frontmatter YAML:

```markdown
---
name: <slug>           # igual ao nome do diretório; vira a invocação /<slug>
description: >         # usado pelo Claude para disparar automaticamente pela intenção do usuário
  Frases de gatilho e descrição da intenção aqui.
---

# Corpo com instruções passo a passo para o Claude seguir
```

## Formato de command

Commands em `commands/<nome>.md` são prompts Markdown simples, sem frontmatter. O nome do arquivo vira o slash command. Wrappers finos apenas delegam:

```markdown
Execute a skill `/nome-da-skill` agora.
```

## Convenções

- **Idioma:** Skills escritas em português (BR).
- **Cabeçalhos de etapa:** Use `## ETAPA N — Descrição` para passos numerados do pipeline.
- **Divisores de seção:** Use `___` (três underscores) entre as etapas.
- **Etapas opcionais:** Marque com `— OPICIONAL SÓ EXECUTE SE HOUVER X CONFIGURADO` no cabeçalho.
- **Etapas críticas:** Marque com `## **IMPORTANTE NUNCA ESQUECER** ETAPA N`.
- **Referências cruzadas entre skills:** Chame outras skills inline como `/nome-da-skill` (ex: `/git-commit-workflow`).
- Não há comandos de build, lint ou testes — este repositório contém apenas Markdown.

## Adicionando uma nova skill

1. Crie `skills/<nome>/SKILL.md` com o frontmatter acima.
2. Escreva o pipeline como seções `## ETAPA` numeradas.
3. Se a skill também deve ser um slash command direto, crie `commands/<nome>.md` delegando para ela.
4. Atualize este arquivo se a nova skill introduzir uma convenção estrutural ainda não documentada aqui.
