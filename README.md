# TS Backend Production Template

Production-ready Node.js + TypeScript Express template with opinionated tooling, security middleware, logging, MongoDB connectivity, migrations, Docker, PM2, and NGINX examples.

## Features
- **Typescript-first** with strict compiler options and source maps
- **Express** app with modular routing and middlewares
- **Security** via Helmet, CORS, and rate limiting
- **Global error handling** and 404 handler
- **Structured logging** with Winston (MongoDB transport configured)
- **MongoDB** with Mongoose
- **Migrations** via `ts-migrate-mongoose` and helper script
- **Dotenv Flow** for multi-env configuration
- **ESLint + Prettier** with TypeScript rules
- **Husky + lint-staged** for pre-commit checks
- **Nodemon** for DX and **PM2** for production process management
- **Docker** setup for development and production
- **NGINX** reverse-proxy examples (HTTP/HTTPS)

## Tech Stack
- Node.js, TypeScript, Express
- Mongoose (MongoDB)
- Helmet, CORS, rate-limiter-flexible
- Winston, winston-mongodb
- dotenv-flow
- ESLint, Prettier, Husky, lint-staged
- PM2
- Docker

## Project Structure
```
src/
  app.ts               # Express app, middleware, routes, error handler
  server.ts            # App bootstrap, DB connect, rate limiter init, startup logs
  router/apiRouter.ts  # Routes: GET /self, GET /health
  controller/          # Controllers (apiController)
  middleware/          # globalErrorHandler, etc.
  service/             # databaseService
  config/              # config, rateLimiter
  util/                # logger, httpError, etc.
public/                # Static assets
docker/                # Dockerfiles (development, production)
nginx/                 # http.conf, https.conf examples
script/migration.js    # Helper for running migrations
ecosystem.config.js    # PM2 configuration
```

## Getting Started

### Prerequisites
- Node.js LTS (recommended)
- MongoDB instance (local or remote)

### Install
```
npm install
```

### Environment Variables
Copy `.env.example` to the appropriate dotenv-flow files. dotenv-flow loads files by `NODE_ENV`.

Minimum variables:
```
# General
ENV=development # production
PORT=3000
SERVER_URL=http://localhost:3000

# Database
DATABASE_URL="mongodb://localhost:27017/your-db"

# Migration
MIGRATE_MONGO_URI="mongodb://localhost:27017/your-db"
MIGRATE_AUTOSYNC="true" # or false
```

### Useful NPM Scripts
- `npm run dev` — Start development mode with nodemon (`NODE_ENV=development`)
- `npm run dist` — Compile TypeScript to `dist/`
- `npm start` — Run compiled app (`NODE_ENV=production`)
- `npm run lint` / `npm run lint:fix` — Lint and auto-fix
- `npm run format:check` / `npm run format:fix` — Prettier check and write
- `npm run migrate:dev` — Run migration helper (development)
- `npm run migrate:prod` — Run migration helper (production)

### Run (Development)
```
npm run dev
```
The app listens on `PORT` (default 3000). Static files served from `public/`.

### Build and Run (Production)
```
npm run dist
npm start
```

## API
Base path: `/api/v1`

- `GET /api/v1/self` — Returns basic service info
- `GET /api/v1/health` — Health check endpoint

## Error Handling
- Centralized error handling via `globalErrorHandler`
- 404 handler converts unknown routes to errors
- `httpError` utility standardizes error responses

## Security
- `helmet()` enabled by default
- `cors()` configured with an allowlist (update `origin` in `src/app.ts`)
- Rate limiting initialized after DB connection via `initRateLimiter`

## Logging
- `winston` logger with support for MongoDB transport (`winston-mongodb`)
- Startup, DB connection, and rate limiter init logs emitted from `server.ts`
- Logs directory and MongoDB logging can be configured in `util/logger`

## Database
- `mongoose` for ODM
- Connection handled in `service/databaseService`
- Provide `DATABASE_URL` in your env

### Migrations
Powered by `ts-migrate-mongoose` with a helper script.

Commands (via helper):
```
# Syntax: node script/migration.js <command> [name]
# Valid commands: create | up | down | list | prune

# examples
node script/migration.js create add-users-collection
node script/migration.js up add-users-collection
node script/migration.js down add-users-collection
node script/migration.js list
node script/migration.js prune
```
Set `MIGRATE_MONGO_URI` (usually same as `DATABASE_URL`).

## Docker

### Development
Dockerfile: `docker/development/Dockerfile`

Example build/run:
```
docker build -f docker/development/Dockerfile -t ts-backend-dev .
docker run --env-file .env -p 3000:3000 ts-backend-dev
```

### Production
Dockerfile: `docker/production/Dockerfile`

Example build/run:
```
docker build -f docker/production/Dockerfile -t ts-backend-prod .
docker run --env-file .env -p 3000:3000 ts-backend-prod
```

## PM2 (Production)
`ecosystem.config.js` runs `dist/server.js` in cluster mode.
```
pm2 start ecosystem.config.js
pm2 status
pm2 logs
```

## NGINX (Reverse Proxy)
Examples in `nginx/http.conf` and `nginx/https.conf`. Point upstream to your app container/host and expose 80/443 accordingly.

## Linting & Formatting
- ESLint config at `eslint.config.mjs` (TypeScript-aware, Prettier-compatible)
- Prettier config at `.prettierrc`
- Pre-commit hooks via Husky + lint-staged for staged `.ts` files

## Troubleshooting
- Ensure `DATABASE_URL` is reachable from the app container/host
- If CORS blocks requests, update the `origin` allowlist in `src/app.ts`
- For rate limit store, ensure DB is connected before requests
- Use `npm run dist` to confirm type safety and build output
- Check `logs/` and PM2 logs for runtime issues

## License
ISC — see `package.json`.