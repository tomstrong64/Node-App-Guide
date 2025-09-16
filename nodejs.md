# Node.js Standards and Practices

## Principles
- Use `"type": "module"` (ESM) in `package.json`.  
- Prefer **native Node.js features** over packages whenever possible  
  (examples: `--env-file` instead of dotenv, `--watch` instead of nodemon).  
  These are **just examples** — always check if Node.js provides a native solution before adding a package.  
- Centralize environment variables in a `config` module.  
- Avoid deprecated packages; prefer minimal dependency trees.  
- Store tool configs in **`package.json`** where supported (e.g. Prettier, ESLint).  
- Application should always be run in Docker (no `dev` or `start` npm scripts).  

---

## package.json (Example)

```json
{
  "type": "module",
  "scripts": {
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "format": "prettier --check .",
    "format:fix": "prettier --write .",
    "test": "vitest",
    "test:watch": "vitest --watch"
  },
  "prettier": {
    "singleQuote": true,
    "semi": true
  }
}
```

---

## Environment Config

`src/config/index.js`

```js
export const config = {
  port: process.env.PORT ?? 3000,
  dbUrl: process.env.DATABASE_URL,
  jwtSecret: process.env.JWT_SECRET
};
```

---

## Checklist
- [ ] `"type": "module"` set.
- [ ] Use native Node.js features where possible.
- [ ] Centralized config module for environment variables.
- [ ] Store configs in `package.json` when supported.
- [ ] No `dev` or `start` scripts — app runs in Docker only.

---

## References
- [Node.js ES Modules](https://nodejs.org/api/esm.html)  
- [Node.js CLI Options](https://nodejs.org/api/cli.html)  
- [NPM Package Search](https://www.npmjs.com/)
