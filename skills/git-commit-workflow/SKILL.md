---
name: git-commit-workflow
description: >
  Executa o pipeline completo de validação e commit de código antes de fazer push para o repositório.
  Use esta skill sempre que o desenvolvedor quiser commitar ou fazer push de alterações, mencionar
  "commitar", "fazer commit", "push", "subir código", "enviar alterações", "git commit", "git push",
  ou quando quiser validar o estado do projeto antes de versionar. A skill garante que o projeto
  compila, testes passam, linters aprovam, não há credenciais expostas e gera uma mensagem de commit
  semântica adequada antes de fazer o push para a branch correta. Também use quando o desenvolvedor
  pedir para "validar o código antes do commit", "checar qualidade do código", ou "preparar release".
---

# Git Commit Workflow

Pipeline completo de validação, revisão e commit semântico de código.

## Visão Geral do Fluxo

```
1. Detectar contexto do projeto
2. Build / Compilação
3. Testes automatizados
4. Linting e análise estática
5. Formatação de código
6. Revisão dos arquivos alterados
7. Varredura de credenciais expostas
8. Validação de impacto das mudanças
9. Geração da mensagem de commit semântico
10. Confirmação da branch com o desenvolvedor
11. Git push
```

___

## ETAPA 1 — Detectar Contexto do Projeto

Antes de qualquer coisa, identifique o ecossistema para usar os comandos certos:

```bash
# Identificar linguagem e ferramentas
ls package.json 2>/dev/null && echo "NODE"
ls pom.xml 2>/dev/null && echo "MAVEN"
ls build.gradle 2>/dev/null && echo "GRADLE"
ls Cargo.toml 2>/dev/null && echo "RUST"
ls go.mod 2>/dev/null && echo "GO"
ls requirements.txt pyproject.toml setup.py 2>/dev/null | head -1 && echo "PYTHON"
ls *.csproj 2>/dev/null && echo "DOTNET"
```

Use a tabela abaixo para mapear o ecossistema aos comandos das etapas seguintes:

| Ecossistema | Build | Testes | Lint | Formato |
|-------------|-------|--------|------|---------|
| Node/TS | `npm run build` ou `npx tsc --noEmit` | `npm test` | `npx eslint .` | `npx prettier --check .` |
| Python | `python -m py_compile **/*.py` | `pytest` | `ruff check .` / `flake8` | `black --check .` |
| Go | `go build ./...` | `go test ./...` | `golangci-lint run` | `gofmt -l .` |
| Rust | `cargo build` | `cargo test` | `cargo clippy` | `cargo fmt --check` |
| Java/Maven | `mvn compile -q` | `mvn test -q` | `mvn checkstyle:check` | — |
| Java/Gradle | `./gradlew build` | `./gradlew test` | `./gradlew checkstyleMain` | — |
| .NET | `dotnet build -q` | `dotnet test` | `dotnet build /warnaserror` | — |

Se o projeto tiver scripts customizados em `package.json` (ex: `npm run lint`, `npm run build`), prefira-os aos comandos genéricos acima.

___

## ETAPA 2 — Build / Compilação

Execute o comando de build do projeto detectado. Capture a saída completa.

- **Se falhar**: pare imediatamente, mostre o erro completo ao desenvolvedor e pergunte se quer corrigir antes de continuar.
- **Se passar**: informe com ✅ e siga para a próxima etapa.

Nunca continue o pipeline se o build falhar.

___

## ETAPA 3 — Testes Automatizados - OPICIONAL SÓ EXECUTE SE HOUVER TESTES CONFIGURADOS

Execute a suíte de testes completa. Reporte:
- Total de testes
- Quantos passaram / falharam / foram pulados
- Detalhes de falhas (nome do teste + mensagem de erro)

- **Se houver falhas**: exiba os testes que falharam e pergunte ao desenvolvedor se quer continuar mesmo assim (pode ser um teste flaky conhecido).
- **Se passar**: informe com ✅.

___

## ETAPA 4 — Linting e Análise Estática - OPICIONAL SÓ EXECUTE SE HOUVER LINTE CONFIGURADO

Execute o linter do projeto. Exemplos por ecossistema:

| Ecossistema | Ferramenta padrão |
|-------------|-------------------|
| Node/TS     | `eslint`, `tsc --noEmit` |
| Python      | `flake8`, `pylint`, `ruff`, `mypy` |
| Rust        | `cargo clippy` |
| Go          | `golangci-lint` |
| Java        | `checkstyle`, `spotbugs` |
| .NET        | `dotnet build /warnaserror` |

Se não houver linter configurado, informe ao desenvolvedor e recomende configurar um.

- **Erros de lint**: exiba os problemas e pergunte se quer corrigir antes de commitar.
- **Apenas warnings**: informe e pergunte se quer commitar assim mesmo.
- **Sem problemas**: ✅

___

## ETAPA 5 — Formatação de Código - OPICIONAL SÓ EXECUTE SE HOUVER FORMATADOR CONFIGURADO

Execute o formatador e verifique se há arquivos que serão modificados:

| Ecossistema | Ferramenta |
|-------------|------------|
| Node/TS     | `prettier --check .` |
| Python      | `black --check .`, `isort --check .` |
| Rust        | `cargo fmt --check` |
| Go          | `gofmt -l .` |
| Java        | `google-java-format --dry-run` |

Se houver arquivos para formatar, **ofereça aplicar a formatação automaticamente** antes de continuar:
> "Encontrei X arquivo(s) fora do padrão de formatação. Deseja que eu aplique a formatação agora?"

___

## ETAPA 6 — Revisão dos Arquivos Alterados e que estão em staged changes

```bash
git diff --staged --stat
git diff --staged
git status
```

Exiba ao desenvolvedor:
1. **Lista de arquivos** modificados, adicionados ou removidos
2. **Diff resumido** das mudanças principais (sem mostrar o diff inteiro se for muito grande — resuma por arquivo)
3. Pergunte: "Há algum arquivo que não deveria estar neste commit?"

Se o desenvolvedor quiser remover algum arquivo do stage:
```bash
git restore --staged <arquivo>
```

___

## ETAPA 7 — Varredura de Credenciais Expostas

Verifique o diff em busca de padrões sensíveis:

```bash
# Verificar no diff staged
git diff --staged | grep -iE \
  "(password|passwd|secret|api[_-]?key|token|private[_-]?key|access[_-]?key|\
  aws[_-]?secret|database[_-]?url|connection[_-]?string|auth[_-]?token|\
  bearer|jwt|-----BEGIN|AKIA[0-9A-Z]{16})" | head -50
```

Também verifique padrões de valores suspeitos (strings longas com alta entropia próximas a chaves):

```bash
git diff --staged | grep -E '=\s*["'"'"'][A-Za-z0-9+/]{20,}["'"'"']' | head -20
```

- **Se encontrar algo suspeito**: **PARE IMEDIATAMENTE** e alerte o desenvolvedor. Não prossiga até o desenvolvedor confirmar que não é uma credencial real, ou até o arquivo ser corrigido.
- **Se limpo**: ✅

___

## ETAPA 8 — Validação de Impacto das Mudanças

Analise o escopo das alterações para classificar o impacto:

```bash
git diff --staged --stat | tail -1  # total de linhas alteradas
git diff --staged --name-only       # arquivos afetados
```

Classifique o impacto e informe ao desenvolvedor:

| Impacto | Critério |
|---------|----------|
| 🟢 Baixo   | < 50 linhas, arquivos isolados, sem mudanças de interface |
| 🟡 Médio   | 50–300 linhas, múltiplos módulos, ou mudanças de API interna |
| 🔴 Alto    | > 300 linhas, mudanças de interface pública, banco de dados, ou breaking changes |

Para impacto **alto**, sugira:
- Dividir em commits menores se possível
- Criar um PR/MR com descrição detalhada
- Notificar o time antes do push

___

## ETAPA 9 — Geração da Mensagem de Commit Semântico

Analise o diff completo e gere a mensagem seguindo o padrão **Conventional Commits**:

### Formato

```
<tipo>(<escopo>): <descrição curta>

[corpo opcional — para mudanças grandes]

[rodapé opcional — BREAKING CHANGE, refs, closes]
```

### Tipos disponíveis

| Tipo | Quando usar |
|------|-------------|
| `feat` | Nova funcionalidade |
| `fix` | Correção de bug |
| `docs` | Apenas documentação |
| `style` | Formatação, sem mudança de lógica |
| `refactor` | Refatoração sem feat nem fix |
| `perf` | Melhoria de performance |
| `test` | Adição ou correção de testes |
| `build` | Mudanças no sistema de build ou dependências |
| `ci` | Mudanças em arquivos de CI/CD |
| `chore` | Tarefas de manutenção, sem impacto no código de produção |
| `revert` | Reversão de commit anterior |

### Regras da descrição curta
- Imperativo, presente: "add feature" não "added feature"
- Sem ponto final
- Máximo 72 caracteres
- Em português ou inglês conforme o padrão do projeto

### Corpo (obrigatório para impacto Médio ou Alto)
Explique **o quê** e **por quê** (não o como). Use linhas de até 72 caracteres.

### Apresente a mensagem ao desenvolvedor e peça confirmação
> "Aqui está a mensagem de commit sugerida: [...]. Quer ajustar algo?"

Permita que o desenvolvedor edite livremente antes de prosseguir.

___

## ETAPA 10 — Seleção da Branch

```bash
# Mostrar branches disponíveis
git branch -a
git branch --show-current
```

Pergunte ao desenvolvedor:
> "Em qual branch você quer fazer o push?
> - Branch atual: `<nome>`
> - Outras branches disponíveis: [lista]"

Se a branch remota não existir ainda, informe e confirme se deve criá-la:
```bash
git push --set-upstream origin <branch>
```

___

## ETAPA 11 — Commit e Push

Com todas as validações aprovadas e mensagem confirmada:

```bash
# Commit
git commit -m "<linha de assunto>" -m "<corpo se houver>"

# Push
git push origin <branch-selecionada>
```

Exiba o resultado do push e confirme com o desenvolvedor:
- URL do commit no repositório remoto (se disponível no output)
- Branch e número de commits enviados

___

## Tratamento de Erros Comuns

| Erro | Ação |
|------|------|
| Push rejeitado (não fast-forward) | Pergunte se quer fazer `git pull --rebase` antes |
| Conflitos de merge | Oriente o desenvolvedor a resolver manualmente |
| Hook de pre-commit falhou | Exiba a saída do hook e pergunte se quer corrigir |
| Sem permissão na branch | Informe e sugira criar um PR/MR para a branch protegida |

___

## Resumo Final

Ao concluir, exiba um resumo do pipeline:

```
✅ Build          — OK
✅ Testes         — 42/42 passaram
✅ Lint           — sem erros
✅ Formatação     — OK
✅ Credenciais    — nenhuma encontrada
🟡 Impacto        — Médio (127 linhas em 5 arquivos)
✅ Commit         — feat(auth): add OAuth2 login with Google
✅ Push           — origin/main ← 1 commit
```