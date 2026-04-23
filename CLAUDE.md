# CLAUDE.md

## Project Overview

A minimal Node.js landing page built with Express. Serves a single static HTML page from the `public/` directory.

- **Language**: JavaScript (Node.js)
- **Framework**: Express 4.x
- **Entry point**: `index.js`
- **Static assets**: `public/`

## Architecture

```
index.js          # Express server — serves public/ as static, root → index.html
public/index.html # Single-page HTML/CSS frontend (no build step)
package.json      # Dependencies: express only
```

## Build & Run

```bash
npm install       # Install dependencies
npm start         # Start server on PORT (default 3000)
PORT=8080 npm start  # Custom port
```

No build step — static files are served directly.

## Tests

No test suite is configured. When adding tests:
- Use `jest` or `node:test` (built-in, no extra dependency)
- Place test files as `*.test.js` alongside the code they test
- Add `"test": "node --test"` or `"test": "jest"` to `package.json` scripts

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
- Inline CSS lives in `public/index.html`; extract to `public/style.css` once it exceeds ~100 lines

## Environment Variables

| Variable | Default | Purpose |
|----------|---------|---------|
| `PORT`   | `3000`  | HTTP listen port |

## Security Notes

- No user input is processed — XSS/injection surface is minimal
- Never commit `.env` files or secrets
- Keep `express` updated; run `npm audit` before releases

## CI/CD

No pipeline configured. If adding GitHub Actions, the standard workflow:
```yaml
- run: npm ci
- run: npm test
- run: npm run lint
```
