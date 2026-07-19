## Project
A minimal REST service (`users`, `health`) backed by an in-memory store.

## Commands

- `npm run dev` — start the API with auto-reload (`node --watch server.js`) on `http://localhost:3000`
- `npm test` — run all tests (Node's built-in test runner, via `node --test`)
- `node --test tests/users.test.js` — run a single test file
- `npm run lint` — check code style with ESLint

CI (`.github/workflows/ci.yml`) runs `npm install`, `npm run lint`, and `npm test` on every push/PR — keep both green.

## Architecture

- `server.js` — entry point; builds the Express `app`, mounts routers, and only calls `app.listen` when run directly (`require.main === module`), so `tests/` can `require("../server")` and drive it with `supertest` against an in-memory instance without opening a real port.
- `routes/` — one file per resource (`users.js`, `health.js`), each exporting an `express.Router()` mounted in `server.js` under its resource path (e.g. `/users`, `/health`).
- `db/store.js` — the only data access layer; routes never touch data directly, they call `store.js` functions (`getAllUsers`, `getUserById`, `createUser`). Data is a plain in-memory array and resets on every restart — there is no real database.
- `tests/` — integration-style tests that hit the Express app through `supertest` rather than unit-testing route handlers in isolation.

## Conventions

- Data flow is strictly routes → `db/store.js`; add new data operations as functions in `store.js` rather than mutating arrays inline in a route.
- Validate required fields and return `400` with `{ error: "..." }` before touching the store (see `POST /users` in `routes/users.js`); return `404` with the same shape when a lookup misses.
- New resources follow the existing pattern: a router file in `routes/`, mounted in `server.js`, with its own section in `tests/`.
- Use eslint to check files syntax
