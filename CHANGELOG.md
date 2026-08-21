# Changelog — schematize-database

Todas as mudanças relevantes deste pacote, no formato [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/),
com versionamento [SemVer](https://semver.org/lang/pt-BR/).


## [0.3.0] — 2026-08-21
Saneamento do catálogo conforme a vistoria de 2026-08-21.

### Corrigido
- **`uuid` no lugar de `bytea` como PK** nos exemplos executáveis de `references/modelo-relacional.md`. Os dois guardam 16 bytes, mas `uuid` é o tipo que o Postgres **entende**: `gen_random_uuid()` como default, ordenação correta e — o que mais importa num incidente — o valor **legível** em `psql`, log e erro (`0190e2…` em vez de `\x0190e2…`). `bytea` como PK ficou como escolha com **motivo escrito**.

### Mudado
- `scripts/check-external-effects.sh` (+ teste) na versão distribuída (md5 idêntico nas 17); `install.sh` regenerado do template; README com a contagem de pisos corrigida (7 → 8).

## [0.2.0] — 2026-08-20

Recorte de **modelagem** do piso "efeito externo NUNCA sai de não-produção"
(`schematize-engineering` → `references/efeitos-externos.md`): e-mail é a única PII **endereçável**
do schema — seed com caixa real é o estopim do disparo em massa que queima IP/domínio e derruba o
**OTP de login** de produção.

### Adicionado
- **SKILL.md** — piso inegociável **8** ("Seed/fixture com e-mail é dado ENDEREÇÁVEL — só o domínio
  de teste em rota nula"), ancorado nos pisos 1 (chave natural `UNIQUE`, nunca PK) e 7 (PII marcada).
- **references/seguranca-pii.md** §6 — "Dado de teste: e-mail de seed/fixture nunca é caixa real":
  vetados (incluindo o `INSERT` de admin dentro da migration), domínio em rota nula, exemplo de
  seed com **`<papel>+<run-id>-<n>@test.<domain>`** preservando o `UNIQUE` (a saída nunca é dropar
  a constraint) e a ressalva de **normalização que remove o `+tag`** (carimbe o run antes do `@`);
  a coluna segue **PII marcada** em qualquer ambiente. Novo item no checklist de design/review.
- **assets/CLAUDE.md** — piso sempre-on **8** com a forma canônica e os vetos.

### Mudado
- **references/seguranca-pii.md** — o checklist de privacidade/segurança passa de §6 para **§7**.

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
