---
name: create-readme
description: >
  Gera ou atualiza o README.md de um repositório com base na análise automática do projeto.
  Use esta skill sempre que o desenvolvedor mencionar o README com intenção de criar, atualizar,
  melhorar ou refletir mudanças nele — independente do nível de detalhe do pedido.
  Exemplos diretos: "cria um README", "gera documentação", "documenta o projeto", "faz o README",
  "preciso de um README", "atualiza o README", "melhora a documentação", "escreve o README para mim",
  "adiciona X no README", "deixa X claro no README", "coloca isso no README".
  Exemplos implícitos (contexto de conversa): "atualiza ele" quando o README foi mencionado antes,
  "reflete isso na documentação", "documenta essa mudança", "o README está desatualizado",
  "adiciona os novos agents/commands/skills no README".
  IMPORTANTE: se o pedido envolve modificar ou reescrever o README.md por qualquer motivo,
  esta skill deve ser invocada — não implemente manualmente.
---

# Create README

Gera um `README.md` completo e profissional para o repositório atual, com base em análise automática
do código, dependências e estrutura do projeto. Sempre confirma informações que não podem ser inferidas
antes de escrever.

## Visão Geral do Fluxo

```
1. Analisar estrutura e ecossistema do projeto
2. Coletar metadados do projeto
3. Explorar funcionalidades e ponto de entrada
4. Verificar configurações e variáveis de ambiente
5. Identificar scripts e comandos disponíveis
6. Perguntar o que não pôde ser inferido
7. Redigir o README
8. Confirmar e salvar
```

___

## ETAPA 1 — Analisar Estrutura e Ecossistema

Mapeie o projeto antes de qualquer outra ação:

```bash
# Estrutura de alto nível (2 níveis de profundidade)
find . -maxdepth 2 -not -path './.git*' -not -path './node_modules*' \
  -not -path './.next*' -not -path './dist*' -not -path './build*' \
  -not -path './__pycache__*' -not -path './target*' \
  | sort

# Detectar ecossistema
ls package.json go.mod Cargo.toml pyproject.toml setup.py requirements.txt \
   pom.xml build.gradle *.csproj composer.json Gemfile 2>/dev/null
```

Identifique:
- **Linguagem principal** — TypeScript, Go, Python, Rust, Java…
- **Framework principal** — Next.js, Fastify, Gin, FastAPI, Spring…
- **Tipo de projeto** — API REST, CLI, biblioteca, frontend, monorepo, serviço de background…
- **Existe README atual?** — `cat README.md 2>/dev/null`

Se existir um README, leia-o inteiro para entender o que já está documentado e o que falta.

___

## ETAPA 2 — Coletar Metadados do Projeto

Extraia as informações estruturadas do manifesto principal:

**Node/TypeScript — `package.json`:**
```bash
# Nome, versão, descrição, scripts, dependências principais
cat package.json
```

**Go — `go.mod`:**
```bash
head -10 go.mod
```

**Python — `pyproject.toml` ou `setup.py`:**
```bash
cat pyproject.toml 2>/dev/null || cat setup.py 2>/dev/null
```

**Rust — `Cargo.toml`:**
```bash
cat Cargo.toml
```

**Java/Maven — `pom.xml`:**
```bash
# Extrair artifactId, groupId, description
grep -E '<(artifactId|groupId|description|version)>' pom.xml | head -10
```

Registre: **nome oficial**, **versão**, **descrição declarada**, **licença**.

___

## ETAPA 3 — Explorar Funcionalidades e Ponto de Entrada

Entenda o que o projeto realmente faz:

```bash
# Ponto de entrada principal
# Node: index.ts / main.ts / server.ts / app.ts
# Go: main.go / cmd/*/main.go
# Python: main.py / app.py / cli.py
# Rust: src/main.rs / src/lib.rs

# Listar rotas HTTP (se aplicável)
grep -rE "(app\.(get|post|put|delete|patch)|router\.(get|post|put|delete|patch)|@(Get|Post|Put|Delete|Patch)|func.*Handler)" \
  --include="*.ts" --include="*.js" --include="*.go" --include="*.py" \
  -h . 2>/dev/null | head -30
```

Leia os 2-3 arquivos mais relevantes (entry point + controller/handler principal) para entender:
- O que o projeto entrega ao usuário final
- Principais recursos/funcionalidades
- Integrações externas (bancos de dados, APIs de terceiros, filas…)

___

## ETAPA 4 — Verificar Configurações e Variáveis de Ambiente

```bash
# Arquivos de configuração e env
ls .env.example .env.sample .env.defaults docker-compose.yml \
   docker-compose.yaml Dockerfile config/ 2>/dev/null

cat .env.example 2>/dev/null || cat .env.sample 2>/dev/null
```

Liste **todas as variáveis de ambiente** encontradas, classificando-as como:
- **Obrigatórias** — o app não sobe sem elas
- **Opcionais** — têm valor padrão ou ativam funcionalidades extras

___

## ETAPA 5 — Identificar Scripts e Comandos Disponíveis

Mapeie como instalar, rodar e testar o projeto:

**Node:**
```bash
# Listar todos os scripts do package.json
jq -r '.scripts | to_entries[] | "\(.key): \(.value)"' package.json 2>/dev/null
```

**Makefile:**
```bash
grep -E '^[a-zA-Z_-]+:' Makefile 2>/dev/null | head -20
```

**Go / Rust / Python:**
```bash
# Comandos de build e execução dos manifestos
```

Identifique os scripts para: **instalar dependências**, **build**, **dev/watch**, **testes**, **lint**, **produção**.

___

## ETAPA 6 — Perguntar o que Não Pôde Ser Inferido

Após a análise, se qualquer um dos itens abaixo não ficou claro pelo código, **pergunte ao desenvolvedor antes de escrever o README**:

- **Propósito do projeto**: O que ele resolve? Para quem?
- **Público-alvo**: Desenvolvedores? Usuários finais? Empresas específicas?
- **Status do projeto**: Em desenvolvimento ativo? Produção? Deprecado? Beta?
- **Pré-requisitos externos**: Conta em algum serviço? Hardware específico?
- **Funcionalidades principais para destaque**: Qual é o diferencial?
- **Idioma do README**: Português, inglês ou bilíngue?
- **Badges desejados**: CI, cobertura de testes, versão, licença?
- **Link do repositório no GitHub/GitLab** (para badges e links)

Faça no máximo **uma rodada de perguntas agrupadas** — não interrompa repetidamente.

Se 80% da informação está clara, escreva o README e sinalize as partes que precisam de revisão
com comentários `<!-- TODO: confirmar com o desenvolvedor -->`.

___

## ETAPA 7 — Redigir o README

Escreva o `README.md` com as seções adequadas ao tipo de projeto. Use o template abaixo como base,
**removendo seções que não se aplicam** e **adicionando seções específicas do projeto**.

### Template Base

```markdown
# Nome do Projeto

> Descrição curta e objetiva em uma linha — o que é e para quem serve.

<!-- Badges opcionais — só inclua se tiver os dados reais -->
![Build](URL_CI_BADGE) ![Versão](URL_VERSION_BADGE) ![Licença](URL_LICENSE_BADGE)

## Sobre

Parágrafo descrevendo o projeto, seu propósito e o problema que resolve.
Inclua o contexto de negócio se relevante.

## Funcionalidades

- ✅ Funcionalidade principal A
- ✅ Funcionalidade principal B
- ✅ Integração com X
- 🚧 Em desenvolvimento: funcionalidade C

## Pré-requisitos

- Node.js >= 20 (ou Go >= 1.22, Python >= 3.11…)
- Docker (opcional, para execução containerizada)
- Conta no serviço X (se necessário)

## Instalação

```bash
# Clonar o repositório
git clone https://github.com/usuario/repositorio.git
cd repositorio

# Instalar dependências
npm install   # ou: go mod download | pip install -r requirements.txt

# Configurar variáveis de ambiente
cp .env.example .env
# Editar .env com os valores adequados
```

## Configuração

| Variável | Descrição | Obrigatório | Padrão |
|----------|-----------|-------------|--------|
| `DATABASE_URL` | URL de conexão com o banco | ✅ | — |
| `PORT` | Porta HTTP do servidor | ❌ | `3000` |
| `LOG_LEVEL` | Nível de log (`debug`, `info`, `error`) | ❌ | `info` |

## Uso

```bash
# Desenvolvimento (com hot-reload)
npm run dev

# Produção
npm run build
npm start
```

### Exemplo de uso (para CLIs ou bibliotecas)

```bash
# Comando de exemplo com output esperado
```

## Testes

```bash
# Todos os testes
npm test

# Com cobertura
npm run test:coverage

# Apenas testes de integração
npm run test:integration
```

## Estrutura do Projeto

```
src/
  controllers/   # Handlers HTTP
  services/      # Lógica de negócio
  repositories/  # Acesso a dados
  models/        # Tipos e schemas
tests/
```

## Contribuindo

1. Fork o repositório
2. Crie uma branch: `git checkout -b feat/minha-feature`
3. Commit suas mudanças: `git commit -m 'feat: adiciona minha feature'`
4. Push para a branch: `git push origin feat/minha-feature`
5. Abra um Pull Request

## Licença

Este projeto está sob a licença [MIT](LICENSE).
```

### Regras de redação

- **Nível de detalhe proporcional à complexidade**: projetos simples têm READMEs curtos.
- **Comandos verificados**: só inclua comandos que existem no projeto (scripts declarados, arquivos presentes).
- **Sem seções vazias**: se não tem testes configurados, não inclua a seção de testes.
- **Exemplos reais**: use nomes de variáveis, endpoints e comandos reais do projeto.
- **Badges**: só inclua se tiver a URL real — badge com URL fake é pior que nenhum badge.

___

## ETAPA 8 — Confirmar e Salvar

Antes de salvar, apresente o README ao desenvolvedor:

> "Aqui está o README gerado. Quer ajustar algo antes de salvar?"

Mostre o conteúdo completo (em markdown) e aguarde confirmação.

Após aprovação (ou se o desenvolvedor disser "salva", "pode salvar", "ok"):

```bash
# Salvar o arquivo
# Use a ferramenta Write para criar/sobrescrever o README.md na raiz do projeto
```

Confirme ao desenvolvedor:
- Caminho do arquivo salvo
- Número de seções incluídas
- Seções marcadas com `<!-- TODO -->` que ainda precisam de revisão

___

## Tratamento de Casos Especiais

| Situação | Ação |
|----------|------|
| README já existe e está bem documentado | Pergunte o que quer atualizar antes de sobrescrever |
| Monorepo com múltiplos pacotes | Gere um README raiz + ofereça gerar READMEs individuais por pacote |
| Projeto privado/interno | Omita instruções de fork/contribuição; foque em onboarding interno |
| Sem licença definida | Não inclua seção de licença; mencione que está faltando |
| Projeto CLI | Inclua seção de referência de comandos com flags e exemplos |
| Biblioteca/SDK | Inclua seção de API com exemplos de uso como módulo importado |
