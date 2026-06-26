---
name: dev-workflow
description: >
  Pipeline completo de desenvolvimento: coleta as tarefas do desenvolvedor (bugs, features, refatorações),
  implementa todas as alterações no código, tira dúvidas quando necessário, e ao final executa o
  git-commit-workflow seguido do git-pr-workflow. Use esta skill quando o desenvolvedor disser "quero
  implementar", "me ajuda a codar", "faz essas mudanças", "preciso corrigir X e implementar Y",
  "desenvolve isso pra mim", "resolve esses bugs", "implementa e já abre o PR", ou qualquer variação
  de pedido de implementação seguida de versionamento. Cobre o ciclo completo: entender → implementar
  → commitar → abrir PR.
---

# Dev Workflow

Pipeline completo de desenvolvimento: entendimento, implementação, commit e abertura de PR.

## Visão Geral do Fluxo

```
1. Levantar contexto do projeto
2. Coletar tarefas com o desenvolvedor
3. Esclarecer dúvidas antes de começar
4. Implementar as alterações
5. Validar implementação com o desenvolvedor
6. Executar git-commit-workflow
7. Executar git-pr-workflow
```

___

## ETAPA 1 — Levantar Contexto do Projeto

Antes de perguntar o que fazer, entenda em que projeto você está:

```bash
# Verificar se é um repositório git
git rev-parse --is-inside-work-tree 2>/dev/null || echo "NOT_A_GIT_REPO"

# Estrutura geral
ls -la
git branch --show-current 2>/dev/null
git log --oneline -5 2>/dev/null

# Linguagem e ecossistema
ls package.json tsconfig.json go.mod Cargo.toml pom.xml requirements.txt pyproject.toml *.csproj 2>/dev/null
```

**Se o diretório não for um repositório git:** alerte o desenvolvedor antes de continuar:

> "Este diretório não é um repositório git. As etapas de commit e PR (Etapas 6 e 7) não serão possíveis sem isso.
> Deseja que eu execute `git init` agora, ou prefere trabalhar apenas na implementação (sem commit/PR)?"

Aguarde a resposta antes de prosseguir. Se o desenvolvedor quiser inicializar:
```bash
git init
git add .
git commit -m "chore: initial commit"
```

### Contexto arquitetural — verifique antes de ler código

Tente ler `docs/architecture/CONTEXT.md`:

- **Se existir:** leia-o. Ele contém a layer map, convenções de nomenclatura, framework
  detectado e interfaces-chave definidos pelo agente arquiteto. Use essas informações
  para guiar onde criar arquivos, como nomeá-los e quais camadas envolver.
  Não é necessário explorar a estrutura de pastas além do que o CONTEXT.md já descreve.

- **Se não existir:** sem problema — siga normalmente. Leia os arquivos abaixo para
  inferir os padrões do projeto:
  - Entry points (`main.ts`, `index.ts`, `main.go`, `app.py`, `Program.cs`, etc.)
  - Arquivos de configuração relevantes
  - CLAUDE.md se existir, para entender o estilo de escrita e padrões do projeto
  - Pastas de módulos/domínios afetados pelas tarefas que serão descritas

Não pergunte o que não precisar: contexto suficiente para entender o projeto já basta.

___

## ETAPA 2 — Coletar Tarefas com o Desenvolvedor

Apresente-se para a sessão e faça as perguntas de levantamento:

> "Olá! Vou te ajudar a implementar e versionar as mudanças. Me conta:
>
> 1. **O que precisa ser feito?**
>    Descreva os bugs a corrigir, as funcionalidades a implementar, ou as refatorações desejadas.
>    Pode listar quantos itens quiser — trabalharei um por um ou em conjunto quando fizer sentido.
>
> 2. **Há algum contexto importante?**
>    Alguma restrição técnica, dependência externa, decisão de design já tomada, ou comportamento
>    que não deve ser alterado."

Aguarde a resposta completa antes de continuar.

### Organizar as tarefas

Após receber o input, monte internamente uma lista priorizada:

```
[ ] Tarefa 1 — <descrição resumida> (tipo: bug / feat / refactor / docs)
[ ] Tarefa 2 — <descrição resumida> (tipo: ...)
...
```

Apresente a lista ao desenvolvedor e confirme se está correta e na ordem certa.

> "Entendi as seguintes tarefas — confirma se está certo e se a ordem faz sentido:
> 1. ...
> 2. ...
> Posso começar?"

___

## ETAPA 3 — Esclarecer Dúvidas Antes de Implementar

Para cada tarefa, identifique o que não está claro antes de tocar no código:

### Tipos de dúvidas que devem ser resolvidas antes:

- **Comportamento esperado ambíguo**: "Quando o usuário faz X, o sistema deve Y ou Z?"
- **Escopo incerto**: "Isso deve afetar apenas o módulo A, ou também o B?"
- **Decisão de design**: "Prefere criar um novo endpoint ou reutilizar o existente?"
- **Dados/contratos**: "O campo `status` pode ser null neste caso?"
- **Dependências externas**: "Já existe uma lib para isso no projeto ou devo implementar do zero?"

### Regras para perguntar:

- Agrupe todas as dúvidas de uma vez por tarefa — não interrompa a implementação com perguntas que poderiam ter sido feitas antes.
- Se uma dúvida surgir durante a implementação e for bloqueante, pause, descreva o problema e pergunte.
- Se a dúvida não for bloqueante, registre internamente e mencione no PR como decisão tomada.

### Formato para perguntar:

> "Antes de implementar a Tarefa 1, preciso esclarecer:
> - [dúvida A]
> - [dúvida B]"

Aguarde as respostas antes de prosseguir.

___

## ETAPA 4 — Implementar as Alterações

Execute as tarefas uma por uma (ou em paralelo quando independentes). Para cada uma:

### 4.1 — Leia os arquivos relevantes antes de editar

```bash
# Encontrar arquivos relacionados à tarefa
find . -name "*.ts" | xargs grep -l "<termo>" 2>/dev/null | head -10
# ou navegue pela estrutura de pastas
```

Sempre leia o arquivo completo antes de editar. Nunca edite às cegas.

### 4.2 — Implemente com qualidade

- Siga os padrões de código existentes no projeto (nomenclatura, estrutura, estilo)
- Não adicione comentários que explicam o óbvio — só comente o que é não-óbvio
- Não adicione tratamento de erro para casos impossíveis
- Não refatore código adjacente que não faz parte da tarefa
- Não adicione abstrações além do necessário para a tarefa atual

### 4.3 — Valide a implementação localmente quando possível

Conforme o ecossistema detectado na Etapa 1:

```bash
# Node/TypeScript
npx tsc --noEmit

# Go
go build ./...

# Rust
cargo check

# Python
python -m py_compile <arquivos>

# Java
mvn compile -q

# .NET
dotnet build --no-restore -q
```

Se houver erros de compilação, corrija antes de avisar o desenvolvedor.

### 4.4 — Informe o progresso

Após completar cada tarefa, informe:

> "✅ Tarefa 1 concluída — `<descrição do que foi feito>`
>    Arquivos alterados: `src/auth/service.ts`, `src/auth/types.ts`"

Se surgirem dúvidas bloqueantes durante a implementação:

> "⏸ Pausei na Tarefa 2: encontrei uma ambiguidade.
>    [descreva o problema]
>    [opção A] vs [opção B] — qual prefere?"

___

## ETAPA 5 — Validar Implementação com o Desenvolvedor

Após completar todas as tarefas, apresente um resumo:

> "Implementação concluída. Resumo do que foi feito:
>
> ✅ **Tarefa 1** — [descrição] → [arquivos alterados]
> ✅ **Tarefa 2** — [descrição] → [arquivos alterados]
>
> Decisões tomadas sem perguntar (não-bloqueantes):
> - [decisão X]: escolhi [opção] porque [razão]
>
> Quer revisar algum arquivo antes de prosseguir para o commit?"

Se o desenvolvedor quiser ajustes, aplique-os agora antes de avançar para as próximas etapas.

___

## ETAPA 5.5 — Atualizar CONTEXT.md — OPCIONAL SÓ EXECUTE SE HOUVER MUDANÇAS ESTRUTURAIS

Execute esta etapa apenas se a implementação criou novos módulos, pastas, interfaces
ou resolveu dúvidas abertas registradas no CONTEXT.md.

### Caso A — CONTEXT.md existe

Atualize **somente** as seções factuais que mudaram:

- **Layer Map** — adicione novas pastas ou módulos criados
- **Key Interfaces / Contracts** — adicione novas interfaces ou tipos centrais
- **Open Questions** — remova as dúvidas que foram respondidas durante a implementação

**Não toque em:** Architecture Style, Naming Conventions, Import Conventions.
Essas seções são domínio exclusivo do agente arquiteto.

Atualize o campo `updated` no frontmatter para a data atual.

### Caso B — CONTEXT.md não existe

Crie `docs/architecture/CONTEXT.md` com uma versão mínima baseada no que foi
observado durante a implementação. Use `unknown — run /architect to fill` para
campos que não foi possível determinar com certeza:

```markdown
---
updated: <YYYY-MM-DD>
scanned_files: 0
---

## Architecture Style
[O que foi observado, ou "unknown — run /architect to fill"]

## Framework
[Nome detectado, ou "unknown — run /architect to fill"]

## Layer Map
[Pastas criadas/observadas → camada correspondente]

## Naming Conventions
unknown — run /architect to fill

## Import Conventions
unknown — run /architect to fill

## Known Circular Dependencies
none detected

## Key Interfaces / Contracts
[Interfaces criadas nesta sessão, se houver]

## Open Questions
- Context created by dev-workflow, not fully analyzed — run /architect to complete
```

___

## **IMPORTANTE NUNCA ESQUECER** ETAPA 6 — Executar git-commit-workflow

Com a implementação aprovada, execute a skill `/git-commit-workflow` completa.

Ela irá:
- Compilar e testar o projeto
- Verificar linting e formatação
- Revisar os arquivos em stage
- Varrer por credenciais expostas
- Gerar a mensagem de commit semântica
- Confirmar a branch e fazer o push

Não pule etapas do `git-commit-workflow` — execute o pipeline completo.

___

## **IMPORTANTE NUNCA ESQUECER** ETAPA 7 — Executar git-pr-workflow

Após o push bem-sucedido, execute a skill `/git-pr-workflow` completa.

Ela irá:
- Coletar o contexto dos commits
- Perguntar o idioma preferido (PT ou EN)
- Gerar o título semântico
- Gerar a descrição estruturada (o que foi feito, por que, como testar, impacto)
- Adicionar a assinatura do autor
- Confirmar tudo com o desenvolvedor
- Criar o PR via `gh pr create`

**Dica de contexto para o git-pr-workflow:** ao gerar a descrição do PR, use as informações levantadas nas Etapas 2–5 (tarefas, decisões de design, arquivos afetados) para enriquecer automaticamente a descrição — o desenvolvedor não precisará repetir o que já explicou.

___

## Tratamento de Situações Especiais

| Situação | Ação |
|----------|------|
| Tarefa muito grande ou ambígua | Quebre em sub-tarefas menores antes de começar |
| Tarefa depende de outra ainda não feita | Implemente na ordem correta e informe a dependência |
| Arquivo não encontrado | Pergunte ao desenvolvedor onde está ou se deve ser criado |
| Conflito entre tarefas (ex: A e B afetam o mesmo trecho) | Resolva o conflito antes de implementar e confirme com o desenvolvedor |
| Tarefa requer mudança de banco de dados | Alerte e pergunte se deve criar a migration |
| Tarefa introduz breaking change | Sinalize claramente e confirme antes de implementar |
| Caso existir muitas tarefas, ou tarefas muito complexas | Sugira dividir em múltiplos commits ou PRs para facilitar revisão |
| Tarefas muito grandes | Divida em sub-tarefas menores e implementar uma de cada vez |

___

## Resumo Final do Pipeline

Ao encerrar, exiba o status completo:

```
✅ Tarefas implementadas  — 3/3
✅ Build                  — OK
✅ Testes                 — OK
✅ Commit                 — feat(auth): add OAuth2 login and fix token refresh
✅ Push                   — origin/feature/oauth2-login
✅ PR aberto              — https://github.com/org/repo/pull/42
```
