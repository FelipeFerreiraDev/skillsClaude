---
name: git-pr-workflow
description: >
  Cria ou atualiza um Pull Request no GitHub com título semântico, descrição estruturada e assinatura
  do autor. Use esta skill sempre que o desenvolvedor quiser abrir um PR, mencionar "abrir PR",
  "criar pull request", "open PR", "criar PR", "subir PR", "mandar PR", ou quando quiser documentar
  as mudanças antes de solicitar revisão. A skill analisa os commits da branch, gera um título
  semântico coerente, monta uma descrição com contexto e motivação, e inclui a assinatura do autor.
  Pergunte ao desenvolvedor se prefere o título e a descrição em português ou inglês antes de criar.
---

# Git PR Workflow

Criação de Pull Request com título semântico, descrição estruturada e assinatura do autor.

## Visão Geral do Fluxo

```
1. Verificar pré-requisitos (gh CLI, remote, branch)
2. Coletar contexto dos commits e diff
3. Definir idioma (PT ou EN) com o desenvolvedor
4. Gerar título semântico
5. Gerar descrição estruturada
6. Confirmar título e descrição com o desenvolvedor
7. Coletar assinatura do autor
8. Criar o PR via gh CLI
9. Exibir resumo final
```

___

## ETAPA 1 — Verificar Pré-requisitos

```bash
# Verificar se gh CLI está disponível
gh --version

# Verificar se há remote configurado
git remote -v

# Verificar branch atual e se há commits não enviados
git branch --show-current
git status --short
git log --oneline origin/HEAD..HEAD 2>/dev/null || git log --oneline -10
```

- Se `gh` não estiver instalado: informe e oriente a instalar com `gh auth login`.
- Se não houver remote: informe que é necessário configurar um repositório remoto.
- Se a branch atual for `main` ou `master`: alerte o desenvolvedor — PRs normalmente partem de branches de feature.

___

## ETAPA 2 — Coletar Contexto dos Commits e Diff

```bash
# Branch atual
BRANCH=$(git branch --show-current)

# Branch base (tenta detectar automaticamente)
git remote show origin | grep "HEAD branch"
git log --oneline origin/HEAD..HEAD 2>/dev/null | head -20

# Diff completo contra a base
git diff origin/HEAD...HEAD --stat
git diff origin/HEAD...HEAD --name-only

# Mensagens de commit desta branch
git log origin/HEAD..HEAD --pretty=format:"%h %s" 2>/dev/null
```

Analise:
1. Quais arquivos foram modificados e em quais módulos/domínios
2. O que as mensagens de commit descrevem
3. Se há breaking changes, migrações de banco, ou mudanças de contrato de API
4. Qual o impacto geral (bugfix, feature, refactor, docs, etc.)

___

## ETAPA 3 — Definir Idioma

Antes de gerar qualquer texto, pergunte ao desenvolvedor:

> "Você prefere o título e a descrição do PR em **português** ou **inglês**?"

Aguarde a resposta e use o idioma escolhido em todas as próximas etapas.

Se o desenvolvedor não tiver preferência, use o mesmo idioma das mensagens de commit da branch.

___

## ETAPA 4 — Gerar Título Semântico

Crie um título seguindo o padrão **Conventional Commits** adaptado para PRs:

### Formato

```
<tipo>(<escopo>): <descrição curta>
```

### Tipos disponíveis

| Tipo       | Quando usar                                              |
|------------|----------------------------------------------------------|
| `feat`     | Nova funcionalidade visível para o usuário               |
| `fix`      | Correção de bug                                          |
| `docs`     | Apenas documentação                                      |
| `style`    | Formatação, sem mudança de lógica                        |
| `refactor` | Refatoração sem nova feat nem fix                        |
| `perf`     | Melhoria de performance                                  |
| `test`     | Adição ou correção de testes                             |
| `build`    | Mudanças no sistema de build ou dependências             |
| `ci`       | Mudanças em arquivos de CI/CD                            |
| `chore`    | Manutenção sem impacto no código de produção             |
| `revert`   | Reversão de mudanças anteriores                          |

### Regras do título

- Imperativo, presente: "add feature" / "adiciona funcionalidade" — não "added" / "adicionado"
- Sem ponto final
- Máximo 72 caracteres
- O `<escopo>` deve ser o módulo, domínio ou caminho mais afetado (ex: `auth`, `users`, `api/orders`)

### Exemplos

```
feat(auth): add OAuth2 login with Google
fix(orders): correct tax calculation for international orders
refactor(users): extract profile update to dedicated service
```

```
feat(auth): adiciona login OAuth2 com Google
fix(pedidos): corrige cálculo de imposto para pedidos internacionais
refactor(usuarios): extrai atualização de perfil para serviço dedicado
```

___

## ETAPA 5 — Gerar Descrição Estruturada

Monte a descrição do PR seguindo esta estrutura:

---

### Em português

```markdown
## O que foi feito

<Explique em 2-4 frases o que foi implementado ou corrigido. Seja direto e objetivo.>

## Por que foi feito

<Explique a motivação: qual problema resolve, qual requisito atende, ou qual melhoria traz.>

## Como testar

- [ ] <Passo 1>
- [ ] <Passo 2>
- [ ] <Resultado esperado>

## Impacto

- **Tipo:** feat / fix / refactor / etc.
- **Módulos afetados:** <lista dos módulos/domínios>
- **Breaking change:** Sim / Não
- **Banco de dados:** Sim (migrations incluídas) / Não

## Notas adicionais

<Opcional: contexto extra, decisões de design, trade-offs, dívida técnica consciente, links relevantes.>
```

---

### In English

```markdown
## What was done

<Explain in 2-4 sentences what was implemented or fixed. Be direct and objective.>

## Why it was done

<Explain the motivation: what problem it solves, what requirement it fulfills, or what improvement it brings.>

## How to test

- [ ] <Step 1>
- [ ] <Step 2>
- [ ] <Expected result>

## Impact

- **Type:** feat / fix / refactor / etc.
- **Affected modules:** <list of modules/domains>
- **Breaking change:** Yes / No
- **Database:** Yes (migrations included) / No

## Additional notes

<Optional: extra context, design decisions, trade-offs, conscious technical debt, relevant links.>
```

---

Se não houver notas adicionais relevantes, omita a seção.

___

## ETAPA 6 — Confirmar com o Desenvolvedor

Apresente o título e a descrição gerados:

> "Aqui está o rascunho do PR:
>
> **Título:** `feat(auth): add OAuth2 login with Google`
>
> **Descrição:**
> [descrição completa]
>
> Quer ajustar algo antes de criar?"

Permita que o desenvolvedor edite livremente. Aplique todas as correções solicitadas antes de avançar.

___

## ETAPA 7 — Coletar Assinatura do Autor

Obtenha as informações do autor configuradas no git local:

```bash
git config user.name
git config user.email
```

Adicione ao **final da descrição** a seguinte assinatura:

```markdown
---
*Authored by: Nome Completo <email@exemplo.com>*
```

Informe ao desenvolvedor qual assinatura será usada e confirme se está correta. Se quiser usar outro nome/e-mail, peça que informe.

___

## ETAPA 8 — Criar o PR via gh CLI

Verifique se a branch já foi enviada ao remote:

```bash
git push --set-upstream origin $(git branch --show-current)
```

Em seguida, crie o PR:

Antes de criar o PR, verifique se o repositório tem labels configuradas:

```bash
gh label list 2>/dev/null | head -20
```

Em seguida, crie o PR:

```bash
gh pr create \
  --title "<título confirmado>" \
  --body "$(cat <<'EOF'
<descrição completa com assinatura>
EOF
)" \
  --base <branch-base> \
  --head $(git branch --show-current) \
  --assignee @me
```

**Labels:** Se o repositório tiver labels e houver uma correspondente ao tipo do PR (`bug`, `enhancement`, `documentation`, `refactor`, etc.), adicione `--label <label>` ao comando. Se não houver labels configuradas ou nenhuma for compatível, **omita o `--label`** — nunca use `--label` com um valor que não exista no repositório, pois o comando falhará.

Opções adicionais — pergunte ao desenvolvedor se quer aplicar:
- `--reviewer <usuario>` → solicitar revisor específico

___

## ETAPA 9 — Exibir Resumo Final

Ao concluir, exiba um resumo:

```
✅ Branch       — feature/oauth2-login → main
✅ Título       — feat(auth): add OAuth2 login with Google
✅ Descrição    — gerada e confirmada
✅ Assinatura   — Felipe Ferreira <email@exemplo.com>
✅ PR criado    — https://github.com/org/repo/pull/42
```

Exiba o link do PR para o desenvolvedor acessar diretamente.

___

## Tratamento de Erros Comuns

| Erro | Ação |
|------|------|
| `gh` não autenticado | Oriente a executar `gh auth login` |
| Branch não enviada ao remote | Execute `git push --set-upstream origin <branch>` antes |
| PR já existe para esta branch | Informe e ofereça atualizar a descrição com `gh pr edit` |
| Branch base incorreta | Pergunte ao desenvolvedor qual deve ser a branch de destino |
| Sem commits à frente da base | Alerte que não há mudanças para abrir PR |
