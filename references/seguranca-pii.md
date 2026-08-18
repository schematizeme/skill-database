# Segurança e privacidade no desenho do schema (PII, LGPD, tenant)

> O piso de **privacidade e segurança que se decide no DESENHO** — antes de qualquer query. Marcar
> PII, dar base legal + retenção, manter PII fora de índice/URL, não guardar segredo em claro, e
> desenhar o **isolamento multi-tenant** na tabela. O que se erra aqui vaza por anos.

Casa com a base (`schematize-engineering` §13-15/§32) e com a governança da **schematize-data**
(`references/governanca.md`): **deny-by-default no acesso**, **nada de PII sem base legal E
retenção**, PII fora de log/URL. Aqui aplicamos esses pisos **na estrutura da tabela**.

## 1. Marque a PII no modelo

Toda coluna que guarda **dado pessoal** (identifica ou torna identificável uma pessoa — nome, email,
CPF, telefone, endereço, IP, geolocalização, foto, dado de saúde/financeiro) é **marcada no desenho**
— comentário na coluna, no resumo do modelo, e no catálogo. O `/database-design` **lista as colunas
PII** na saída; o `/database-review` **aponta PII não marcada** como achado.

Marcar não é enfeite — é o que habilita retenção, mascaramento, export/eliminação por titular (LGPD)
e o gate de "essa coluna pode ir pra índice/URL/log?".

## 2. Base legal + retenção — obrigatórias

Nenhuma PII entra no schema sem **duas** respostas (piso da casa, LGPD):

- **Base legal:** por que a casa pode tratar esse dado (consentimento, execução de contrato,
  obrigação legal, legítimo interesse…). Sem base legal, a coluna **não existe**.
- **Retenção:** por quanto tempo, e o que acontece depois (apagar ou anonimizar). Toda PII **expira**
  — o desenho prevê **como** (coluna `deleted_at`/`anonymized_at`, partição por tempo que se dropa,
  rotina de expurgo). "Guarda pra sempre por via das dúvidas" é **VETADO**.

Registre base legal + retenção junto da coluna (catálogo/ADR) — é o que a auditoria e a
`schematize-data` consomem.

## 3. PII fora de índice em claro, de URL e de chave natural exposta

- **PII em claro NÃO é chave de índice pesquisável.** Índice de email/CPF em claro é vazamento por
  desenho (dump do índice = dump da PII; buscas revelam presença). Se precisa **buscar** por email,
  indexe um **hash determinístico** (com pepper) ou trate o identificador de forma controlada
  (`citext` só quando o dado já é identificador público). Busca por CPF idem.
- **PII nunca na URL/query string** (`schematize-engineering` §16.1/§32) — e portanto **nunca** como
  a chave que a API expõe. A chave exposta é o **id surrogate** (`normalizacao-chaves-indices.md`
  §3); email/CPF/slug-com-nome ficam como atributo interno.
- **Identidade ≠ email.** A PK da pessoa é o id surrogate interno; o email é atributo `UNIQUE`
  mutável — nunca a PK, nunca o que aparece em rota. (Piso do IAM da casa.)
- **id sequencial exposto** revela volume e permite enumeração — VETADO, use ULID/UUIDv7
  (`normalizacao-chaves-indices.md` §3; a `schematize-pentest` ataca exatamente isso: IDOR/BOLA,
  enumeração).

## 4. Segredo nunca em claro — e o que NÃO guardar

O banco de **modelagem** não guarda credencial reutilizável em claro:

- **Senha:** só **hash** forte (argon2id) — nunca a senha, nunca hash reversível. (O fluxo de auth é
  do IAM; aqui, a **coluna** é `senha_hash`, não `senha`.)
- **Token/API key/secret:** hash ou referência a cofre — não o valor. Dado de cartão/PAN **não** se
  guarda (tokenização/gateway).
- **Minimização:** só as colunas que o domínio **precisa**. Coluna PII "que talvez sirva" é
  superfície de vazamento sem uso — não a crie. Menos PII guardada, menos a proteger.

## 5. Isolamento multi-tenant — no desenho, não no `WHERE`

Sistema multi-tenant nasce com o isolamento **na estrutura**, não confiado a um `WHERE` que alguém
pode esquecer (`schematize-engineering` §15, `schematize-data` §governança):

- **`tenant_id` na tabela** (toda tabela de dado de tenant) e **na frente dos índices quentes**
  (`normalizacao-chaves-indices.md` §5) — alinha isolamento e performance.
- **`tenant_id` nas chaves compostas** quando o fato é único **por tenant** (`unique (tenant_id,
  codigo)`, não `unique (codigo)` global).
- **Deny-by-default + RLS quando couber:** Row-Level Security (Postgres) força o filtro de tenant no
  **banco**, derivado do contexto server-side — não do cliente. O `tenant_id` **nunca** vem confiado
  do cliente; vem do token validado server-side (a **query parametrizada** que aplica isso é da skill
  de linguagem — SQL concatenado é VETADO lá).
- **FK respeita o tenant:** uma FK não deve permitir apontar pra linha de **outro** tenant — modele
  a chave/constraint pra que cross-tenant seja impossível por estrutura (ex.: FK composta incluindo
  `tenant_id`).

## 6. Checklist de privacidade/segurança (design/review)

- [ ] Toda coluna PII **marcada** (comentário + resumo + catálogo).
- [ ] Cada PII tem **base legal** e **política de retenção** (com o mecanismo de expurgo desenhado).
- [ ] **Nenhuma PII em claro** como chave de índice pesquisável, em URL, ou como chave natural
      exposta.
- [ ] Chave exposta é o **id surrogate** ULID/UUIDv7; **identidade≠email**; sem id sequencial
      exposto.
- [ ] Segredo (senha/token/cartão) **nunca em claro** — hash/cofre/tokenização; coluna é
      `*_hash`/referência.
- [ ] **Minimização** — só as colunas que o domínio precisa.
- [ ] Multi-tenant: `tenant_id` na tabela e nos índices; unicidade **por tenant**; deny-by-default
      (RLS quando couber); FK não cruza tenant.
