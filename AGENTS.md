# ofx-br

Biblioteca Python que lê arquivos OFX de bancos brasileiros: SGML 1.x com tags sem
fechamento, encoding cp1252 com cabeçalho contraditório e deduplicação de
lançamentos por FITID. Zero dependências em runtime, só a biblioteca padrão.

## Stack e estrutura

- Python >= 3.10, build com `hatchling`, sem dependências de runtime.
- `src/ofxbr/`: pacote publicado (`parser.py`, `sgml.py`, `encoding.py`, `values.py`,
  `dedup.py`, `models.py`, `errors.py`, `__main__.py` para a CLI `ofxbr`).
- `tests/`: `test_ofxbr.py` e `make_fixtures.py` (gerador das fixtures em
  `tests/fixtures/*.ofx`).

## Comandos obrigatórios

- Setup: `pip install -e ".[dev]"`
- Lint: `python -m ruff check .`
- Formatação: `python -m ruff format --check .`
- Typecheck: não há (projeto não usa mypy/pyright).
- Test: `python -m pytest -q --cov=ofxbr --cov-report=term-missing`
- Fixtures: `python tests/make_fixtures.py && git diff --exit-code -- tests/fixtures`
  (fixture editada à mão sem passar pelo gerador vira teste que só passa na
  máquina de quem editou)
- Build: não há comando de build no CI (empacotamento via `hatchling` fica para
  o momento de publicação, fora do escopo de CI).
- E2E: não há.

Esses são os mesmos passos do `.github/workflows/ci.yml`, rodados na matriz
ubuntu/windows x Python 3.10 a 3.13.

## Convenções de arquitetura e segurança específicas

- Todo valor monetário é `decimal.Decimal`, nunca `float` (ver `values.py`).
  Ponto flutuante binário acumula erro de centavos em conciliação com milhares
  de lançamentos.
- Quando o arquivo não declara fuso horário, o `datetime` resultante fica
  ingênuo (sem `tzinfo`). Não inventar fuso (`America/Sao_Paulo` ou outro) por
  conta própria: isso pode jogar o lançamento para o dia errado.
- Encoding é decidido pelo `CHARSET` do cabeçalho OFX, não pelo `ENCODING`
  (que costuma declarar `USASCII` mesmo quando o corpo é cp1252). Cabeçalho
  ausente cai em cp1252 por padrão, e esse fallback nunca deve levantar
  exceção de decodificação.
- O parser SGML (`sgml.py`) precisa aceitar tag folha com e sem fechamento no
  mesmo arquivo; não assumir XML bem formado para OFX 1.x.
- `dedupe`/`find_duplicates`/`new_since` operam por FITID; o modo
  `use_fingerprint=True` existe para bancos que reemitem FITID na transição
  pendente -> efetivada, e compara `(data, valor, histórico normalizado)`
  ignorando hora e sequências longas de dígitos. Não ligar esse modo por
  padrão: gera falso positivo em compras legítimas do mesmo valor, mesmo dia,
  mesmo estabelecimento.
- Investimento (`INVSTMTRS`) e mensagens fora de `STMTRS`/`CCSTMTRS` não são
  suportados; não expandir escopo sem pedido explícito.

## Áreas críticas

- `tests/fixtures/*.ofx`: gerado por `tests/make_fixtures.py`. Nunca editar à
  mão; o CI falha se o conteúdo divergir do gerador.
- API pública em `src/ofxbr/__init__.py`: é o contrato consumido via
  `pip install ofx-br`. Mudança de assinatura ou remoção é breaking change.
- `pyproject.toml` (`[project]`, `classifiers`, `dependencies`): não adicionar
  dependência de runtime sem justificativa forte, o projeto existe para ser
  zero-dependência.

## Regras de trabalho

- Mudanças pequenas e no escopo pedido. Sem commit, push ou deploy sem pedido
  explícito: entregas passam pelo fluxo `/entrega`.
- Convenções genéricas (idioma pt-BR, Ponytail, segurança, worktrees, rubricas
  de outcome, revisão cruzada) vivem nos arquivos globais
  (`~/.claude/CLAUDE.md` e `~/.codex/AGENTS.md`) e não são repetidas aqui.

## Formato de entrega

Ao final de uma fatia, reportar: arquivos alterados, decisões não óbvias
tomadas, comandos de validação executados com resultado, e pendências (com a
mensagem de erro original, se houver).

## Decisões do projeto

Ainda não existe `decisions.md` neste repositório. Ao registrar a primeira
decisão arquitetural relevante, criar `.claude/memory/decisions.md` no formato
ADR e apontar para ele aqui.
