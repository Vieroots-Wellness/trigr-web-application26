# trigr-web-application26

Umbrella repository that bundles the Limoverse backend (`backend/`) and frontend (`frontend/`) as git submodules tracking the `trigr` branch of each upstream.

## Repository layout

```
trigr-web-application26/
├── backend/    →  Vieroots-Wellness/limoverse @ trigr   (Laravel 12 / PHP 8.2)
└── frontend/   →  Vieroots-Wellness/limoverse_frontend @ trigr   (React Router 7 / Vite)
```

## 1. Clone

Clone with submodules in one shot:

```bash
git clone --recurse-submodules -b develop git@github.com:devvieroots/trigr-web-application26.git
cd trigr-web-application26
```

Already cloned without submodules?

```bash
git submodule update --init --recursive
```

## 2. Configure the backend (`backend/`)

The backend ships with a Docker Compose stack (PHP-FPM, Nginx, Postgres 16, Redis 7, Horizon, MailPit, MinIO, pgAdmin) and a Makefile wrapping common tasks.

```bash
cd backend

# Copy env template and edit credentials / ports as needed
cp .env.example .env

# One-shot interactive setup (configures ports + credentials, builds, installs)
make setup-full

# Or step by step:
make up                  # start containers
make install             # composer install + npm install + key:generate
make migrate             # run migrations
make seed                # seed demo data (optional)
```

Useful commands:

| Command | Purpose |
|---|---|
| `make shell` | Open a shell in the app container |
| `make logs` | Tail all container logs |
| `make fresh` | Drop DB + re-migrate + re-seed |
| `make test` | Run the Pest test suite in Docker |
| `make ps` | Show running containers |
| `make down` | Stop everything |

Quality gate (must pass before pushing on the backend):

```bash
docker compose exec app composer check   # Pint + PHPStan level 8 + Pest with 100% coverage
```

See `backend/CLAUDE.md` for full architecture conventions (DDD, 4-layer Clean Architecture, TDD rules).

## 3. Configure the frontend (`frontend/`)

```bash
cd frontend
npm install

# Pick the environment file you want
cp .env.development .env       # local dev
# cp .env.staging .env         # to point at staging APIs

npm run dev                    # http://localhost:3000 (react-router dev)
```

Other scripts:

| Command | Purpose |
|---|---|
| `npm run build` | Production build (optimizes images, builds with Vite, tracks bundle size) |
| `npm run start` | Serve the production build |
| `npm run typecheck` | `react-router typegen && tsc` |
| `npm run test` | Vitest in watch mode |
| `npm run test:coverage` | Vitest with coverage |
| `npm run test:e2e` | Playwright end-to-end tests |
| `npm run lighthouse` | Lighthouse CI run |

## 4. Working with the submodule pointers

Each submodule is pinned to a specific commit on the `trigr` branch. To pull the latest `trigr` tips into this umbrella repo:

```bash
# from the umbrella repo root
git submodule update --remote --merge
git add backend frontend
git commit -m "chore: bump backend + frontend submodule pointers"
git push
```

To work on one of the submodules directly:

```bash
cd backend     # or frontend
git checkout trigr        # detach from the pinned commit
# ...make changes, commit, push to origin/trigr...
cd ..
git add backend           # record the new submodule SHA
git commit -m "chore(backend): advance to <short-sha>"
git push
```

## 5. Required tools

| Tool | Version |
|---|---|
| Docker + Docker Compose | for the backend stack |
| Node.js | ≥ 20 (frontend) |
| npm | ≥ 10 (frontend) |
| Git | ≥ 2.34 (submodule support) |

The backend's PHP / Composer / Postgres / Redis all run inside Docker — no host install needed beyond Docker itself.
