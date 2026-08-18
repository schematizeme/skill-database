# Changelog — schematize-database

Formato: [Keep a Changelog](https://keepachangelog.com/pt-BR/). Versionamento semântico.

## [0.1.0] — 2026-08-18

Primeira versão da skill de **modelagem de banco de dados** da casa — 100% dedicada ao **DESENHO
do schema relacional em si** (schema design), o passo anterior à implementação. Desdobra o §10
(Banco de Dados) da `schematize-engineering` numa disciplina própria de desenho, com fronteira
nítida: a engenharia de **DADOS** (pipeline/ETL/CDC/contratos) é da `schematize-data`; a
**implementação** (queries parametrizadas, ORM, rodar migration) é das skills de linguagem.

### Adicionado
- **SKILL.md** com 7 pisos inegociáveis (PK surrogate ULID/UUIDv7 + natural `UNIQUE`,
  identidade≠email; normalizar até 3FN, desnormalizar registrado; tipo é a 1ª constraint, sem
  "texto pra tudo"; integridade é do banco com constraints; índice é decisão de custo, PII fora de
  índice/URL; migration expand-contract reversível, nunca `DROP`/rename num passo; PII marcada com
  base legal + retenção, deny-by-default + tenant no desenho) + mapa de references + a seção
  **"o que esta skill NÃO faz (fronteira)"** com data e linguagens + relação com as outras skills.
- **references/**:
  - `modelo-relacional.md` — domínio→schema: entidade→tabela, atributo→coluna atômica (1FN),
    relações 1:1/1:N/N:N com **tabela de junção**, cardinalidade/obrigatoriedade, o **tipo correto
    por coluna** (dinheiro sem `float`, tempo `timestamptz` UTC, enum/CHECK, jsonb com parcimônia)
    e a ordem de emitir o DDL.
  - `normalizacao-chaves-indices.md` — 1FN–3FN e anomalias, desnormalização medida+registrada,
    **chaves** (surrogate ULID + natural `UNIQUE`, identidade≠email, id sequencial VETADO),
    **constraints** (`NOT NULL`/`UNIQUE`/`CHECK`/`FK`+tabela de `ON DELETE`), **índices** (FK,
    compostos e ordem igualdade→range, parcial, cobertura, redundância, PII fora do índice) +
    checklist de desenho.
  - `migrations.md` — **expand→migrate→contract** reversível (`up`/`down` testado), os casos
    clássicos (rename/tipo/NOT NULL/FK em tabela grande sem travar via `CONCURRENTLY`/`NOT VALID`),
    pisos, **particionamento** básico (RANGE tempo/retenção, LIST/HASH tenant) e o que o design
    emite.
  - `seguranca-pii.md` — **PII marcada** + base legal + retenção (LGPD), PII fora de índice/URL/chave
    exposta, identidade≠email, segredo nunca em claro, minimização, **isolamento multi-tenant** no
    desenho (`tenant_id`, unicidade por tenant, RLS, FK que não cruza tenant) + checklist.
- **assets/commands/**: `/database-help`, `/database-design` (projeta schema da descrição do
  domínio → `CREATE TABLE` + migration expand-contract + resumo do modelo), `/database-review`
  (revisa schema existente), `/database-load`, `/database-claude`, `/database-cc`,
  `/database-handoff`.
- **assets/CLAUDE.md** — regra sempre-on de modelagem: chaves surrogate + natural, 3FN,
  constraints no banco, migration expand-contract, PII marcada; e a fronteira (dados→data,
  implementação→linguagem).
