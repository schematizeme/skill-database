# schematize-database

> **Modelagem de banco de dados** da casa (schema design) — a disciplina normativa, **agnóstica de
> linguagem**, do **DESENHO do schema relacional em si**, antes de uma linha de código de aplicação.
> Um schema mal desenhado é dívida que **todo** o resto herda: query lenta, dado inconsistente,
> migration impossível, PII vazando. Esta skill é o **piso do desenho** — o modelo antes do código.

Pacote de **skill normativa para [Claude Code](https://claude.com/claude-code)**.
Parte do catálogo **schematize skills**. Desdobra o **§10 (Banco de Dados)** da
`schematize-engineering` numa disciplina própria de **desenho de schema**, com fronteira nítida:
a engenharia de **DADOS** (pipeline/ETL/CDC/contratos) é da `schematize-data`; a **implementação**
(queries parametrizadas, ORM, rodar migration) é das skills de linguagem.

## Instalar

### Pelo app schematize (recomendado)

```bash
schematize install database   # requer o CLI schematize instalado
```

### Última versão (a partir de um clone)

```bash
git clone https://github.com/schematizeme/skill-database.git
cd skill-database && ./install.sh            # instala no projeto atual
# ./install.sh /caminho/do/projeto            # ou aponte para outro projeto
# ./install.sh ~                              # global (~/.claude, todos os projetos)
```

Ou baixe o `.zip` da última release e descompacte em `.claude/skills/`:

```bash
curl -L -o skill-database.zip \
  https://github.com/schematizeme/skill-database/releases/latest/download/skill-database.zip
unzip skill-database.zip -d .claude/skills/
```

## Comandos

| Comando | O que faz |
|---|---|
| `/database-help` | lista todos os comandos do schematize-database |
| `/database-design` | **projeta um schema a partir da descrição em linguagem natural do domínio**: entidades→tabelas/colunas/tipos/PK/FK/índices seguindo os padrões da casa, e **emite o SQL (`CREATE TABLE`) + a migration expand-contract reversível + um resumo do modelo** (entidades, relações, chaves, PII) — pronto pro "database builder" do app consumir |
| `/database-review` | **revisa um schema existente** (DDL/migration/`\d`): normalização (1FN–3FN, redundância), chaves (surrogate + natural), tipos, constraints/FK (`ON DELETE`), índices (faltando/redundante/PII), particionamento e PII/LGPD — aponta achados e emite as migrations de correção |
| `/database-load` | carrega à força TODO o corpo normativo (modelo relacional, normalização/chaves/índices, migrations, segurança/PII) e passa a aplicá-lo |
| `/database-claude` | cria ou mescla o `CLAUDE.md` sempre-on de modelagem na raiz do repo |
| `/database-cc` | context compact: gera handoff no archive e roda `/compact` |
| `/database-handoff` | gera o handoff (context.md + checklist.md) sem compactar |

## O que tem dentro

- **SKILL.md** — o contrato: 8 pisos inegociáveis (PK surrogate ULID/UUIDv7 + chave natural
  `UNIQUE`, identidade≠email; normalizar até 3FN, desnormalizar registrado; o tipo é a 1ª
  constraint, sem "texto pra tudo"; integridade é do banco com constraints; índice é decisão de
  custo e PII fora de índice/URL; migration expand-contract reversível, nunca `DROP`/rename num
  passo; PII marcada com base legal + retenção, deny-by-default + tenant no desenho) + mapa de
  references + a seção "o que esta skill NÃO faz (fronteira)".
- **references/** — `modelo-relacional` (domínio→schema: entidade→tabela, atributo→coluna atômica,
  relações 1:1/1:N/N:N com **tabela de junção**, cardinalidade/obrigatoriedade e o **tipo correto
  por coluna**), `normalizacao-chaves-indices` (1FN–3FN e anomalias, desnormalização
  medida+registrada, **chaves** surrogate ULID + natural `UNIQUE`, **constraints**
  `NOT NULL`/`UNIQUE`/`CHECK`/`FK`+`ON DELETE`, **índices** compostos/ordem/parcial/cobertura),
  `migrations` (**expand→migrate→contract** reversível, casos clássicos, **particionamento**
  básico) e `seguranca-pii` (**PII marcada** + base legal + retenção LGPD, identidade≠email,
  isolamento multi-tenant no desenho).
- **assets/commands/** — `/database-help`, `/database-design`, `/database-review`,
  `/database-load`, `/database-claude`, `/database-cc`, `/database-handoff`.
- **assets/CLAUDE.md** — regra sempre-on do piso de modelagem.

## Regra de ouro

**O schema é o contrato mais duradouro do sistema.** O código muda toda semana; a tabela sobrevive
anos e carrega o dado real. Por isso o desenho é **plan-first** e **conservador na integridade** —
o banco é a **última linha de defesa** da consistência. "Depois a gente ajusta o schema" é caro:
ajustar schema com dado vivo é migration com risco; ajustá-lo **no desenho** é grátis. PK é
surrogate interna (ULID/UUIDv7), a chave natural é `UNIQUE`; normalize até 3FN; o tipo é a primeira
constraint; a integridade é do **banco**; migration é expand-contract reversível; PII é marcada.

## Relação com as outras skills

- **schematize-engineering** — a **BASE**. Esta skill desdobra o **§10 (Banco de Dados)** e casa
  com os pisos de dados/eventos, segurança/multi-tenancy/LGPD, archive (§28) e DoD (§35).
- **schematize-data** — a **irmã de dados**, com fronteira nítida: aqui é o **desenho do schema
  relacional (OLTP)**; lá é o **dado como contrato** em movimento (pipeline/CDC/streaming/qualidade/
  lineage/warehouse). O **expand-contract reversível** é o piso que as duas compartilham.
- **schematize-go / rust / elixir / csharp / zig / ruby** — a **implementação**: query
  parametrizada, ORM, executar migration, testes de repositório. O schema é o mesmo em qualquer stack.
- **schematize-pentest** — o **oráculo ofensivo**: id sequencial exposto (enumeração/IDOR),
  cross-tenant por falta de `tenant_id`/RLS, PII vazando em índice/coluna.

Co-autoria / patrocínio: Lucassa — https://lucassa.me

MIT.
