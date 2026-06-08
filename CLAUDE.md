# CLAUDE.md

## Project Overview

A minimal Node.js landing page built with Express. Serves a single static HTML page from the `public/` directory.

- **Language**: JavaScript (Node.js)
- **Framework**: Express 4.x (intentionally pinned — do not upgrade to 5.x without an explicit task)
- **Entry point**: `index.js`
- **Static assets**: `public/`

## Architecture

```
index.js          # Express server — serves public/ as static, root → index.html
public/index.html # Single-page HTML/CSS frontend (no build step)
package.json      # Dependencies: express only
```

### Route pattern

`index.js` mounts `express.static('public')` for all static assets AND registers an explicit `app.get('/')` that sends `public/index.html`. The static middleware alone would serve the root, but the explicit route provides a reliable `__dirname`-anchored path that is safe across working-directory changes. **Do not remove the explicit route.** New routes go below the static middleware mount.

### Modules already imported in `index.js`

`require('express')` and `require('path')` are already present at the top of `index.js`. Do not add duplicate `require()` calls for either. Before adding any new `require()`, scan the existing imports to avoid duplication.

## Build & Run

```bash
npm install       # Install dependencies
npm start         # Start server on PORT (default 3000)
PORT=8080 npm start  # Custom port
node --check index.js  # Parse syntax without starting the server
```

No build step — static files are served directly.

`package-lock.json` is not currently committed. **The first PR that modifies `package.json` dependencies must include `package-lock.json`.** Always run `npm install` after changing dependencies and stage the lockfile — `npm ci` (used in CI) requires it and will fail without it.

## Tests

No test suite is configured. When adding tests:
- Use `jest` or `node:test` (built-in, no extra dependency)
- Place test files as `*.test.js` alongside the code they test
- Add `"test": "node --test"` or `"test": "jest"` to `package.json` scripts
- Before writing any test, update `index.js`: add `module.exports = app` and guard `app.listen` behind `if (require.main === module)`. Both changes are required — missing either breaks `supertest` imports or causes the server to start during test runs.

**Current state**: `index.js` does not yet export `app` and does not guard `app.listen`. The first PR that introduces any test file must also make both of these changes to `index.js`, otherwise the test process will bind the port and hang.

## Linting

No linter configured. Recommended setup:
```bash
npm install --save-dev eslint
npx eslint --init
```
Add `"lint": "eslint ."` to scripts.

## Coding Conventions

- CommonJS (`require`/`module.exports`) — do not switch to ESM without updating `package.json` `"type"`
- `const` by default; `let` only when reassignment is needed
- No comments unless the WHY is non-obvious
- Keep `index.js` thin — route handlers that grow beyond ~20 lines belong in a `routes/` or `controllers/` directory
- Inline CSS lives in `public/index.html`; extract to `public/style.css` once it exceeds ~100 lines (currently ~53 lines)

## Error & 404 Handling

Express matches middleware by parameter count. An error-handling middleware **must** declare exactly 4 parameters `(err, req, res, next)` — a 3-parameter signature is silently treated as regular middleware and errors bypass it entirely.

Both a 404 catch-all and an error handler must be mounted **after** all route definitions:

```js
// After all app.get / app.post / etc.
app.use((req, res) => res.status(404).send('Not found'));         // 404 — 3 params
app.use((err, req, res, next) => {                               // error — 4 params
  console.error(err);
  res.status(500).send('Internal server error');
});
```

Neither handler is present in `index.js` yet. When adding them, order matters: 404 handler before error handler, both after all routes.

## Environment Variables

| Variable | Default | Purpose |
|----------|---------|---------|
| `PORT`   | `3000`  | HTTP listen port |

## Security Notes

- No user input is processed — XSS/injection surface is minimal
- Never commit `.env` files or secrets
- Keep `express` updated; run `npm audit --audit-level=high` before releases — PRs adding dependencies must be clean at `high` severity
- When adding user-facing endpoints: set `Content-Security-Policy` and `X-Content-Type-Options` headers
- Never use `path.join` on user-supplied input without a path-traversal guard
- `res.sendFile` must always use an absolute path anchored to `__dirname` — never a relative path or user-supplied value
- `express.static` must use `path.join(__dirname, 'public')`, not a bare `'public'` string — a bare string resolves from `process.cwd()`, which differs from `__dirname` if the server starts from another directory. The existing `express.static('public')` in `index.js` is a known deviation; correct it if that line is touched
- Never call `next()` on the same code path as `res.send`/`res.json`/`res.end` — this causes a double-response crash
- Use `express.json()` only on routes that need it; avoid `app.use(express.json())` globally unless all routes require a body
- Always pass a `limit` option to `express.json()` / `express.urlencoded()` (e.g., `{ limit: '10kb' }`) to prevent unbounded request bodies

## Commit & PR Conventions

Commit messages follow **Conventional Commits**:

```
<type>: <short description>
```

Common types: `feat`, `fix`, `chore`, `refactor`, `docs`, `test`.

PR titles must match the same format. If the work closes a GitHub issue, include `Closes #NNN` in the PR description body (not the title). GitHub auto-closes the issue on squash-merge, and the issue number appears in the resulting commit (`(#NNN)`).

Example PR title: `feat: add /healthz endpoint`
Example PR body line: `Closes #42`

## CI/CD

No pipeline configured. If adding GitHub Actions, the standard workflow:
```yaml
- run: npm ci
- run: npm test
- run: npm run lint
```
