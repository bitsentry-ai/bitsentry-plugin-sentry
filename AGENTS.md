# Sentry plugin

`@bitsentry/plugin-sentry` is a BitSentry code plugin that reads Sentry organizations, projects, issues and issue events. BitSentry loads it through `@bitsentry/plugin-sdk`, so its descriptor and action results are a public contract.

## What we never compromise on

- **Read-only.** Every action is `riskLevel: "read"`, including the OAuth helpers (`build_authorize_url`, `exchange_code_for_token`, `refresh_token`). The plugin never changes Sentry data. Do not add a write action without a maintainer asking for it.
- **Credentials stay in auth.** Auth tokens, access tokens, refresh tokens and client secrets come from host-managed auth or the OAuth flow. Never log them, return them in error summaries, or put them in persisted configuration.
- **Correct results.** A partial page or a rate-limited request must be reported as such, never as a complete empty result.

## The ways to hurt yourself

1. **Calling real Sentry.** Never run the plugin against a real Sentry organization unless the developer asks. Tests stub `fetch`.
2. **Unbounded retries.** Retries honor `Retry-After` and stop at a fixed number of attempts, inside the host's `operation.signal` and `operation.deadlineAt`. Keep them bounded.
3. **Breaking the contract.** Renaming an action id, field key or result field breaks saved runbooks and hosts. Add, do not rename.

## Verifying

CI runs these. Run them before you push:

```sh
pnpm install --frozen-lockfile
pnpm run format
pnpm run lint
pnpm test
pnpm run build
pnpm pack --dry-run
pnpm knip
```

Behavior changes ship with a focused test in `src/plugin.spec.ts`. Bug fixes get a failing test first.

## Pull requests

- Never open a PR unless the developer asks. Branch from the latest `origin/main`.
- Titles and every commit subject are lowercase conventional commits: `feat(sentry): honor parent operation cancellation`. `change-guardrails.yml` checks them.
- Body: `## Problem`, `## Change`, `## Tests`.
- Pushing a `v*` tag publishes a GitHub release. Keep `version` in `package.json` and the descriptor in step.
- Do not add `Co-Authored-By` lines or AI attribution footers.

## Where code lives

- `src/plugin.ts`: descriptor, auth fields, OAuth helpers, actions and the Sentry API client.
- `src/plugin.spec.ts`: Vitest tests.
- `scripts/eslint/`: the TypeScript 6 lint bridge.

## Taste

- Fail loudly and specifically. An empty result must mean "nothing found", not "the request failed".
- Comments say why. Plain language in summaries. No em-dashes.
