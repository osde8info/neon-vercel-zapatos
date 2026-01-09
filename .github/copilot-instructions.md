<!-- Short, actionable instructions for AI coding agents working on this repo -->
# Copilot instructions for neon-vercel-zapatos

This project is a small Vercel Edge Function demo that uses Zapatos (typed DB layer) with Neon’s serverless Postgres driver. The goal of this file is to give an AI coding agent the exact, actionable context needed to be productive quickly.

- **Big picture:** Vercel Edge runtime exposes a Request handler in `api/sites.ts` which creates a `Pool` from `@neondatabase/serverless`, queries the DB via Zapatos, and returns JSON of nearby UNESCO sites based on IP geolocation.

- **Key files & why they matter:**
  - `api/sites.ts` — primary Edge function; shows how Zapatos `db.select`, `db.sql` and `db.param` are used together. Inspect here to understand query patterns and how compiled SQL can be logged (see the commented `query.compile()` lines).
  - `shims/pg/*` — this local package is published as `pg` in `package.json` and simply re-exports `@neondatabase/serverless`. This is intentional: Zapatos imports `pg` and must resolve to Neon’s serverless driver in a serverless environment.
  - `update-zapatos-types.mjs` — development script that runs `zapatos/generate` to create `zapatos/schema.d.ts`. It also sets `neonConfig.webSocketConstructor = ws` because Node lacks a built-in `WebSocket`.
  - `zapatos/schema.d.ts` — generated types: do NOT edit. If schema types are stale, run `npm run update-zapatos-types` (see `package.json` script).
  - `package.json` — contains `update-zapatos-types` script and dependency mapping of `pg` to `shims/pg`.

- **Developer workflows / commands (exact):**
  - Install deps: `npm install`
  - Update Zapatos types from DB: `npm run update-zapatos-types` (requires `DATABASE_URL` in `.env.local` — see README instructions using `npx vercel env` and `npx vercel env pull .env.local`)
  - Run locally: `npx vercel dev`
  - Deploy: `npx vercel deploy`
  - Load example data: use the `psql` client against `DATABASE_URL` as shown in `README.md`.

- **Project-specific conventions / gotchas for agents:**
  - Preserve `shims/pg` behavior: do not replace or remove this shim; Zapatos expects to import `pg`.
  - `zapatos/schema.d.ts` is generated. If you need to change schema, update DB and regenerate types via the provided script; don’t hand-edit the file.
  - `update-zapatos-types.mjs` depends on `ws` and sets `neonConfig.webSocketConstructor`. If you change the generator flow, keep this detail so schema generation succeeds on Node.
  - `tsconfig.json` must include the `zapatos/**/*` path and use `strict: true` (project depends on the generated types being included in TypeScript builds).
  - Edge runtime config is set in `api/sites.ts` (`export const config = { runtime: 'edge', regions: [...] }`) — changing this affects deployment behavior.

- **Examples to copy/paste when modifying or adding DB queries:**
  - Use Zapatos SQL fragments for geometry operations, e.g. in `api/sites.ts` the `distance` fragment is built as:
    - `const distance = db.sql<s.whc_sites_2021.SQL, number>`${"location"} <-> st_makepoint(${db.param(longitude)}, ${db.param(latitude)})`;
  - Compose an `extras` column and ordering with Zapatos `db.select` as shown in `api/sites.ts`.

- **When editing code, check these places first:**
  - `api/sites.ts` for runtime and query shape
  - `update-zapatos-types.mjs` for schema generation changes
  - `shims/pg/*` and `package.json` for dependency mappings

- **Testing / validation tips for agents:**
  - To verify schema regeneration: ensure `DATABASE_URL` points at a DB containing the expected schema, then run `npm run update-zapatos-types` and confirm `zapatos/schema.d.ts` is updated.
  - To inspect produced SQL for a Zapatos query: uncomment the `query.compile()` block in `api/sites.ts` and run `npx vercel dev` to see the SQL and param list in server logs.

If anything in this file is unclear or you want the agent to include more examples, tell me which areas to expand (e.g., more Zapatos examples, local dev setup, or deployment notes).
