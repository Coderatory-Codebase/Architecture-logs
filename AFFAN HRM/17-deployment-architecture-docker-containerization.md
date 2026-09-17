## 17. Deployment Architecture --- Docker & Containerization

Both apps get containerized for consistent local dev and reproducible
production deploys --- matching the `.dockerignore` already present in
the backend.

### Backend --- `base_server/Dockerfile`

    # Stage 1: build
    FROM node:20-alpine AS builder
    WORKDIR /app
    COPY package*.json ./
    RUN npm ci
    COPY . .
    RUN npm run build

    # Stage 2: production runtime
    FROM node:20-alpine AS runner
    WORKDIR /app
    ENV NODE_ENV=production
    COPY package*.json ./
    RUN npm ci --omit=dev
    COPY --from=builder /app/dist ./dist
    EXPOSE 3000
    CMD ["node", "dist/bin/server.js"]

### Frontend --- `calendar-frontend/Dockerfile`

    # Stage 1: deps
    FROM node:20-alpine AS deps
    WORKDIR /app
    COPY package*.json ./
    RUN npm ci

    # Stage 2: build
    FROM node:20-alpine AS builder
    WORKDIR /app
    COPY --from=deps /app/node_modules ./node_modules
    COPY . .
    RUN npm run build

    # Stage 3: production runtime
    FROM node:20-alpine AS runner
    WORKDIR /app
    ENV NODE_ENV=production
    COPY --from=builder /app/public ./public
    COPY --from=builder /app/.next/standalone ./
    COPY --from=builder /app/.next/static ./.next/static
    EXPOSE 3001
    CMD ["node", "server.js"]

*(Requires* `output: 'standalone'` *in* `next.config.ts` *for the
minimal* `runner` *stage above.)*

### Local development --- root-level `docker-compose.yml`

    version: '3.8'
    services:
      backend:
        build: ./base_server
        ports:
          - '3000:3000'
        env_file:
          - ./base_server/.env
        depends_on:
          - mongo

      frontend:
        build: ./calendar-frontend
        ports:
          - '3001:3001'
        depends_on:
          - backend

      mongo:
        image: mongo:7
        ports:
          - '27017:27017'
        volumes:
          - mongo-data:/data/db

    volumes:
      mongo-data:

`mongo` here is a **local dev-only override** --- production still
points `DATABASE_URL` at MongoDB Atlas as it does today; the local
container just means a new contributor can `docker compose up` without
needing their own Atlas cluster to start developing. Neon Postgres stays
cloud-hosted in both dev and prod (it's serverless --- no meaningful
benefit to containerizing it locally).

### Production targets

  -----------------------------------------------------------------------
  Component               Target                  Reasoning
  ----------------------- ----------------------- -----------------------
  Frontend                Vercel                  Native Next.js support,
                                                  zero-config, matches
                                                  the app already being
                                                  Next.js

  Backend                 Render / Railway /      Any of these run the
                          Fly.io                  `Dockerfile` above
                                                  directly; pick based on
                                                  free-tier limits during
                                                  the portfolio phase

  MongoDB                 Atlas (existing)        Already in use, no
                                                  change

  PostgreSQL              Neon (existing)         Already in use, no
                                                  change

  CI/CD                   GitHub Actions          On every PR:
                                                  `npm run lint` →
                                                  `npx tsc --noEmit` →
                                                  `npm test` →
                                                  `npm run build`, for
                                                  **both** repos ---
                                                  nothing merges if any
                                                  step fails
  -----------------------------------------------------------------------
