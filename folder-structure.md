# Project Folder Structure Standards

## Folder Tree

```text
/                        # Project root
├── docs/                # Documentation
├── src/                 # Application source code
│   ├── bin/             # Entrypoints (e.g. index.js)
│   ├── config/          # Centralized configuration
│   ├── constants/       # Application-wide constants
│   ├── controllers/     # Request controllers
│   ├── errors/          # Custom error classes/handlers
│   ├── middleware/      # Express middlewares
│   ├── models/          # Database models
│   ├── repositories/    # Data access layer
│   ├── routes/          # Express route definitions
│   ├── services/        # Business logic
│   ├── utils/           # Utility/helper functions
│   └── app.js           # App bootstrap
├── test/                # Unit and integration tests
├── package.json         # Package and tooling config
├── .env                 # Environment variables (never committed)
└── README.md
```

---

## Rules
- All source code lives in `/src`.  
- Entrypoint = `/src/bin/index.js`.  
- Use plural folder names (`controllers`, `services`, etc.).  
- Test files live in `/test`.  
- `/docs` for API + external documentation.  

---

## Checklist
- [ ] Root dirs: `docs`, `src`, `test`.  
- [ ] Clear separation: controllers, services, repositories.  
