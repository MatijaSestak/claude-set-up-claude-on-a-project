# CLAUDE.md

It's the first hands-on project of a Claude Code course from Mate Academy.

## Commands

- `npm run dev`: start the API with `node --watch` on `http://localhost:3000` (`PORT` overrides it)
- `npm test`: run all tests with Node's built-in runner (`node --test`)
- `node --test --test-name-pattern="404" tests/users.test.js`: run a single test by name
- `npm run lint`: ESLint (`eslint:recommended`, ES2022)

CI (`.github/workflows/ci.yml`, Node 22) runs `npm run lint` and then `npm test` on every push and PR, so run both before committing.

## Conventions

- Use CommonJS (`require` / `module.exports`), not ES modules. ESLint is set to `sourceType: "script"`.
- Write tests with `node:test` and `node:assert`, and send HTTP requests with `supertest` against the exported `app`. Don't add Jest or Mocha.
- Route handlers never touch the `users` array directly. All reads and writes go through functions exported from `db/store.js`; add a new store function when you need a new operation.
- Errors are JSON `{ "error": "<message>" }` with the matching status code (400 for validation, 404 for not found). A successful create returns 201.
- Route `:id` params arrive as strings. Convert them with `Number(...)` before calling the store, because the store compares ids with `===`.

## Architecture

- `server.js` builds the Express app, mounts each router under its path prefix (`/users`, `/health`) and exports `app`. It calls `listen()` only when it is run directly (`require.main === module`), so tests can import `app` without opening a port.
- `routes/` has one `express.Router` file per resource. A new resource gets its own file plus an `app.use(...)` line in `server.js`.
- `db/store.js` is module-level in-memory state that stands in for a real database. It resets on restart. All test files in one process share that state, so tests that create users change what later tests see. Don't assert exact list lengths or ids.
