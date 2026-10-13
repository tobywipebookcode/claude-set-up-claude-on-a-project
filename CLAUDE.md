# CLAUDE.md

A small Express REST API (users + health check) backed by an in-memory store

## Commands

- `npm run dev` — start the API with `node --watch` on http://localhost:3000 (port from `PORT`)
- `npm test` — run all tests with Node's built-in runner (`node --test`)
- `node --test --test-name-pattern="404" tests/users.test.js` — run a single test by name
- `npm run lint` — ESLint over the whole repo

CI (`.github/workflows/ci.yml`, Node 22) runs `npm run lint` then `npm test`; both must pass.

## Conventions

- CommonJS only: use `require`/`module.exports`, not `import`/`export` (ESLint is set to `sourceType: "script"`).
- Tests use `node:test` + `node:assert` + `supertest`, not Jest/Mocha. Import `app` from `../server` and call `request(app)` — never start a real server in tests.
- Route handlers never touch data directly; all reads/writes go through functions exported from `db/store.js`.
- Errors are returned as JSON `{ error: "<message>" }` with the matching status code (400 for validation, 404 for missing).
- Unused `req`, `res`, `next`, or `_` params are allowed by the lint config; don't rename or remove them to silence warnings.
- Never read or commit `.env`; config examples belong in `.env.example`.

## Architecture

- `server.js` builds the Express app, mounts each router under its resource path (`/users`, `/health`), and exports `app`. It only calls `listen()` when run directly (`require.main === module`), which is what lets tests import it.
- `routes/` holds one `express.Router()` file per resource. Adding a resource means a new file here plus an `app.use(...)` line in `server.js`.
- `db/store.js` is a module-level in-memory array standing in for a database. State resets on restart but is **shared across tests in the same process** — tests that create users will affect later assertions, so don't rely on exact counts or IDs.
