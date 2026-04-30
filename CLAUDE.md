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

## Build & Run

```bash
npm install       # Install dependencies
npm start         # Start server on PORT (default 3000)
PORT=8080 npm start  # Custom port
```

No build step — static files are served directly.

`package-lock.json` is not currently committed. If you run `npm install` and a lockfile is generated, commit it so that `npm ci` produces a reproducible install in CI.

## Tests

No test suite is configured. When adding tests:
- Use `jest` or `node:test` (built-in, no extra dependency)
- Place test files as `*.test.js` alongside the code they test
- Add `"test": "node --test"` or `"test": "jest"` to `package.json` scripts
- Before writing any test, update `index.js`: add `module.exports = app` and guard `app.listen` behind `if (require.main === module)`. Both changes are required — missing either breaks `supertest` imports or causes the server to start during test runs.

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

## Environment Variables

| Variable | Default | Purpose |
|----------|---------|---------|
| `PORT`   | `3000`  | HTTP listen port |

## Security Notes

- No user input is processed — XSS/injection surface is minimal
- Never commit `.env` files or secrets
- Keep `express` updated; run `npm audit` before releases
- When adding user-facing endpoints: set `Content-Security-Policy` and `X-Content-Type-Options` headers
- Never use `path.join` on user-supplied input without a path-traversal guard
- `res.sendFile` must always use an absolute path anchored to `__dirname` — never a relative path or user-supplied value
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
