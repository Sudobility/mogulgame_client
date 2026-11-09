# @sudobility/mogulgame_client

API client SDK for MogulGame, with TanStack Query hooks. TypeScript, ESM, built and
run with [Bun](https://bun.sh). Published to **GitHub Packages** (`npm.pkg.github.com`).

## Installation

This package lives in the GitHub Packages registry under the `@sudobility` scope, so a
consuming project needs an `.npmrc` that points the scope at it:

```
@sudobility:registry=https://npm.pkg.github.com
```

Then install (Bun is the supported package manager):

```bash
bun add @sudobility/mogulgame_client
```

Peer dependencies you must also install:

```bash
bun add react @tanstack/react-query @sudobility/types
```

`@sudobility/mogulgame_types` is a type-only dependency used by this package's public
types. It is a devDependency here (so the package compiles), but consumers must install
it themselves:

```bash
bun add @sudobility/mogulgame_types
```

## Usage

Everything is exported from the package root — there are no subpath entry points.

```ts
import {
  StarterClient,            // HTTP client class (name is template lineage; see note below)
  usePropertySearch,
  useProperty,
  useOffers,
  useFavorites,
  QUERY_KEYS,
} from "@sudobility/mogulgame_client";

// Create the HTTP client. `networkClient` is dependency-injected — this package
// never calls fetch directly.
const client = new StarterClient({ baseUrl: "https://api.example.com", networkClient });

// In a React component. Hooks take networkClient and baseUrl as the first two args
// (parameter-based DI — no provider/context):
const { data, isLoading } = usePropertySearch(networkClient, baseUrl, params);
const { offers, createOffer, updateOffer, cancelOffer } = useOffers(networkClient, baseUrl, token);
```

> **The exported class is named `StarterClient`, not `MogulGameClient`.** This is template
> lineage from the `starter_client` project. Consumers typically alias it on import
> (`import { StarterClient as MogulGameClient }`). Renaming it is a breaking change.

## API

### `StarterClient`

HTTP client class, constructed with `{ baseUrl, networkClient }`. It uses dependency
injection via the `NetworkClient` interface from `@sudobility/types` — no direct `fetch`
calls. Methods map 1:1 to `mogulgame_api` endpoints:

`getUser`, `searchProperties`, `getProperty`, `getPropertyHistory`, `getOffers`,
`getOffer`, `createOffer`, `updateOffer`, `cancelOffer`, `getUserProfile`,
`getTransactions`, `getLeaderboard`, `getPopularProperties`, `favoriteProperty`,
`unfavoriteProperty`, `getFavorites`, `checkFavorites`.

### Hooks

All hooks take `(networkClient, baseUrl, ...)` and build a memoized `StarterClient`
internally.

- `usePropertySearch`, `useProperty`, `usePropertyHistory` — property search / detail / price history
- `useOffers` — user offers query plus `createOffer` / `updateOffer` / `cancelOffer` mutations
- `useUserProfile`, `useTransactions` — current user profile and transaction history
- `useLeaderboard` — rankings (by balance or wins, scoped by country)
- `usePopularProperties` — popular properties feed
- `useFavorites` — favorites query plus `favorite` / `unfavorite` mutations
- `useFavoriteCheck` — check whether a set of property IDs are favorited

### Utilities & constants

- `createAuthHeaders(token)` / `createHeaders(...)` — build request headers
- `buildUrl(base, path)` — URL construction helper
- `handleApiError(error)` — standardized API error handling
- `QUERY_KEYS` — type-safe TanStack Query cache key factory (all keys namespaced under `'mogulgame'`)
- `DEFAULT_STALE_TIME` (5 min) / `DEFAULT_GC_TIME` (30 min) — default query cache timings
- `FirebaseIdToken` — branded `string` type for Firebase ID tokens

## Development

Install dependencies first (`bun install`), then:

```bash
bun run build          # Compile ESM (tsc) + fix relative import extensions for Node ESM
bun run build:watch    # tsc in watch mode
bun run clean          # Remove dist/
bun run typecheck      # TypeScript check (tsc --noEmit)
bun run lint           # ESLint over src/
bun run lint:fix       # ESLint with --fix
bun run format         # Prettier write over src/
bun run format:check   # Prettier check
bun run test:unit      # Run Vitest once (colocated *.test.ts files)
bun run test:watch     # Vitest in watch mode
bun run verify         # typecheck + test:unit + build (run before committing)
```

> There is no `test` script — use `bun run test:unit` (or `test:watch`). The `test:unit`
> script invokes `vitest` from `node_modules/.bin`, so run `bun install` first.

The build runs `scripts/fix-esm-imports.mjs` after `tsc` to rewrite extensionless relative
import specifiers (`./core` → `./core/index.js`) so the emitted ESM resolves under Node's
native loader.

## CI/CD

`.github/workflows/ci-cd.yml` delegates to the shared reusable workflow
`johnqh/workflows/.github/workflows/unified-cicd.yml`, which publishes to GitHub Packages
(`publish-target: github`) on pushes to `main` / `develop`.

## Related Packages

- **mogulgame_types** — shared type definitions (`Property`, `PretendOffer`, request/response types) and runtime helpers
- **mogulgame_api** — backend server this SDK talks to over HTTP
- **mogulgame_lib** — pure helpers (does not import this package, and this package does not import it)
- **mogulgame_app** — web frontend; imports these hooks directly
- **mogulgame_app_rn** — React Native app; depends on the published package

The consumer supplies the `NetworkClient` implementation, allowing a different one per
platform (e.g. a Firebase-auth network service on web).

## License

BUSL-1.1
