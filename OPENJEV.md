# OpenJEV support

This fork adds optional support for [OpenJEV](https://openjev.sh), a free community gateway to
the same Jev model that TypeSafe serves. OpenJEV is technology only — a public API endpoint. It
is not a replacement for TypeSafe; TypeSafe stays the default and anyone with a TypeSafe key sees
zero behaviour change.

Jev is built by [TypeSafe](https://typesafe.ai).

## What was added

- `guideme/src/scalars.rs` — `Model::openjev()`, the model id `openjev` for the OpenJEV gateway.
- `guideme/src/api/client.rs` — `OPENJEV_DEFAULT_BASE_URL` constant (`https://api.openjev.sh`),
  `503` added to the retryable statuses alongside `429`/`529`, and provider-neutral retry log
  messages.
- `guideme/src/api/mod.rs` — re-exports `OPENJEV_DEFAULT_BASE_URL`.
- `guideme/src/guide.rs` — `GuideBuilder::from_env` now selects a provider; `GuideBuilder::provider`
  setter; `provider` field on the guide so the `guideme.ask` span records `gen_ai.provider.name`
  as `openjev` or `typesafe`.
- `README.md` — OpenJEV note after the intro, env-var guidance, configuration and errors tables
  updated.

No TypeSafe code path was renamed, removed or re-defaulted.

## Provider selection rule

1. `JEV_PROVIDER=openjev` → OpenJEV (explicit choice wins).
2. Otherwise, if `TYPESAFE_API_KEY` is set → TypeSafe, exactly as before (default unchanged).
3. Otherwise, if only `OPENJEV_API_KEY` is set → OpenJEV.

## How to configure

TypeSafe (default, unchanged):

```sh
export TYPESAFE_API_KEY=...
```

OpenJEV:

```sh
export OPENJEV_API_KEY=...
# or, with a TypeSafe key also present, force OpenJEV:
export JEV_PROVIDER=openjev
export OPENJEV_API_KEY=...
```

Optional overrides: `OPENJEV_BASE_URL` (default `https://api.openjev.sh`), `GUIDEME_MODEL`
(overrides the model for either provider).

## Wire contract

Same request/response shape as TypeSafe (`POST /v1/systemone`):

| | TypeSafe direct | OpenJEV |
|---|---|---|
| Endpoint | `https://api.typesafe.ai/v1/systemone` | `https://api.openjev.sh/v1/systemone` |
| Model | `jev-latest` | `openjev` |
| Key | `TYPESAFE_API_KEY` | `OPENJEV_API_KEY` |
| Retryable | `429`, `529` | `429`, `503`, `529` |

## How it was verified

A live `POST https://api.openjev.sh/v1/systemone` with model `openjev`, state `ping` and one
noul question returned HTTP 200. No repository code was executed during the port.

## Upstream

Original project: https://github.com/pedro-pscunha/guideme-rust by @pedro-pscunha.
