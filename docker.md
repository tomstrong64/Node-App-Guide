# Docker Standards for Node.js Applications

## Principles
- Use Docker for both dev and prod.  
- Pin Node.js to exact LTS image (e.g. `node:22.20.0-alpine3.22`).  
- Use Alpine images when possible.  
- Multi-stage builds:
  - **dev** → all deps, hot reload with `--watch`.  
  - **prod** → prod deps only, `dumb-init`, non-root.  
- Always run as `USER node`.  
- Use `dumb-init` in prod.

---

## Example Dockerfile

```dockerfile
FROM node:22.20.0-alpine3.22 AS base

# ---- Development ----
FROM base AS dev
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm ci
COPY . .
USER node
EXPOSE 3000
CMD ["node", "--watch", "src/bin/index.js"]

# ---- Production ----
FROM base AS prod
RUN apk add --no-cache dumb-init
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm ci --include=production
COPY . .
USER node
EXPOSE 3000
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "src/bin/index.js"]
```

---

## docker-compose (Dev)

```yaml
services:
  app:
    build:
      context: .
      target: dev
    ports:
      - "3000:3000"
    volumes:
      - .:/usr/src/app
      - /usr/src/app/node_modules
    env_file:
      - .env
```

---

## .dockerignore

```text
node_modules
Dockerfile
compose.yml
.env
.git
.gitignore
```

---

## Checklist
- [ ] Pin exact Node.js LTS + Alpine.  
- [ ] Dev installs all deps.  
- [ ] Prod installs only prod deps.  
- [ ] Run as non-root `node`.  
- [ ] Use `dumb-init` in prod.  
- [ ] Exclude secrets/unneeded files in `.dockerignore`.

---

## References
- [Docker Docs](https://docs.docker.com/)
- [Node.js Docker Hub](https://hub.docker.com/_/node)
- [dumb-init](https://github.com/Yelp/dumb-init)  
- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
