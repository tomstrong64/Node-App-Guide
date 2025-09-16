# Styling and Linting Standards

## Principles
- Use **Prettier** for formatting.  
- Use **ESLint** with **Airbnb Base Rules**.  
- Use `eslint-config-prettier` to disable conflicting rules.  
- Store configuration in `package.json` wherever supported.  
- Run lint/format via `npm run lint`, `npm run lint:fix`, `npm run format`, `npm run format:fix`.  

---

## Dependencies

Install Airbnb ESLint config and its peer dependencies automatically:

```bash
npx install-peerdeps --dev eslint-config-airbnb-base
npm install --save-dev prettier eslint-config-prettier
```

---

## Example package.json Config

```json
{
  "scripts": {
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "format": "prettier --check .",
    "format:fix": "prettier --write ."
  },
  "eslintConfig": {
    "extends": [
      "airbnb-base",
      "prettier"
    ],
    "env": {
      "node": true,
      "es2022": true
    }
  },
  "prettier": {
    "singleQuote": true,
    "semi": true,
    "printWidth": 80
  }
}
```

---

## Checklist
- [ ] Installed Airbnb config via `npx install-peerdeps`.  
- [ ] Prettier + eslint-config-prettier installed.  
- [ ] ESLint uses Airbnb base rules.  
- [ ] Prettier overrides conflicting ESLint rules.  
- [ ] Config stored in `package.json`.  
- [ ] Scripts available: `lint`, `lint:fix`, `format`, `format:fix`.  

---

## References
- [ESLint Docs](https://eslint.org/docs/latest/use/configure)  
- [Airbnb Base Config](https://www.npmjs.com/package/eslint-config-airbnb-base)  
- [install-peerdeps](https://www.npmjs.com/package/install-peerdeps)  
- [Prettier Docs](https://prettier.io/docs/en/options.html)
