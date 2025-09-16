# Testing Standards

## Principles
- Use **Vitest** for unit and integration testing.  
- Use **Supertest** for HTTP API tests.  
- Use **mongodb-memory-server** for a fully portable MongoDB test environment.  
- All tests live in `/test`.  
- Tests should be runnable entirely in Docker.  

---

## Dependencies

```bash
npm install --save-dev vitest supertest mongodb-memory-server
```

---

## Vitest Config

`vitest.config.js`

```js
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    environment: "node",
    globals: true,
    coverage: {
      reporter: ["text", "json", "html"]
    }
  }
});
```

---

## Example Test (health check)

`test/health.test.js`

```js
import request from "supertest";
import app from "../src/app.js";

test("Health check works", async () => {
  const res = await request(app).get("/health");
  expect(res.status).toBe(200);
  expect(res.body).toEqual({ status: "ok" });
});
```

---

## Example MongoDB Memory Server Usage

`test/db.test.js`

```js
import { MongoMemoryServer } from "mongodb-memory-server";
import mongoose from "mongoose";

let mongod;

beforeAll(async () => {
  mongod = await MongoMemoryServer.create();
  const uri = mongod.getUri();
  await mongoose.connect(uri);
});

afterAll(async () => {
  await mongoose.disconnect();
  await mongod.stop();
});
```

---

## Checklist
- [ ] Use Vitest for all tests.  
- [ ] Use Supertest for API route testing.  
- [ ] Use mongodb-memory-server for MongoDB tests (no external DB dependency).  
- [ ] Tests stored in `/test`.  
- [ ] Coverage reporting enabled.  
- [ ] Tests runnable in Docker.  

---

## References
- [Vitest Docs](https://vitest.dev/guide/)  
- [Supertest Docs](https://github.com/visionmedia/supertest)  
- [mongodb-memory-server](https://github.com/nodkz/mongodb-memory-server)
