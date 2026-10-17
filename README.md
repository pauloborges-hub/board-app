# Board App

A full-stack feedback board inspired by [Featurebase](https://www.featurebase.app). Users browse product ideas and bug reports on a Kanban-style board, open an issue to read the details, like it to show interest and discuss it in the comments.

## Features

- **Kanban board** with four columns: Backlog, To-do, In progress and Done.
- **Search** issues from the header; the query is kept in the URL (`?q=`).
- **Issue details** page with description, status, likes and comments. From the board it opens as a modal (Next.js intercepting + parallel routes). Visiting `/issues/[id]` directly renders the full page, with a dynamic Open Graph image for link previews.
- **Likes** on issues, with the like count and your own like state loaded separately from the cached board data.
- **Comments** on issues (create, edit and delete through the API).
- **Authentication** with GitHub OAuth. Liking and commenting require a signed-in user.
- **REST API** with auto-generated OpenAPI spec and interactive docs.
- **Caching** of issue and comment lists with Next.js Cache Components, invalidated on writes (for example when a comment is created).

## Tech stack

| Area | Technology |
| --- | --- |
| Framework | [Next.js 16](https://nextjs.org) (App Router, Cache Components) and [React 19](https://react.dev) |
| Language | TypeScript |
| Styling | [Tailwind CSS 4](https://tailwindcss.com), [Radix UI](https://www.radix-ui.com) Dialog, [Lucide](https://lucide.dev) icons |
| API | [Hono](https://hono.dev) with [`@hono/zod-openapi`](https://github.com/honojs/middleware/tree/main/packages/zod-openapi), served from a Next.js route handler |
| API docs | [Scalar](https://scalar.com) API reference |
| Database | PostgreSQL 17 |
| ORM and migrations | [Drizzle ORM](https://orm.drizzle.team) and Drizzle Kit |
| Auth | [Better Auth](https://www.better-auth.com) (GitHub provider) |
| Data fetching and forms | [TanStack Query](https://tanstack.com/query), [React Hook Form](https://react-hook-form.com), [nuqs](https://nuqs.dev) |
| Validation | [Zod 4](https://zod.dev) |
| Tooling | pnpm, ESLint, Docker Compose, [Faker](https://fakerjs.dev) (seed data) |

## Project structure

```
src/
├── api/                  # Hono API (runs inside Next.js)
│   ├── index.ts          # App setup: CORS, Better Auth, OpenAPI docs, routes
│   ├── auth.ts           # Better Auth configuration
│   ├── middlewares/      # requireAuth middleware
│   ├── routes/           # One file per endpoint (zod-openapi route + handler)
│   └── db/               # Drizzle client, schema, migrations and seed script
├── app/                  # Next.js App Router
│   ├── (board)/          # Board page (home) and header with search
│   ├── issues/[id]/      # Issue details page
│   ├── @modal/           # Parallel route that renders issues as a modal
│   └── api/[[...route]]/ # Mounts the Hono app at /api
├── components/           # Shared UI components
├── http/                 # Server and client functions that call the API
├── lib/                  # Auth client and React Query provider
└── env.ts                # Client-side environment variables
```

## Getting started

### Prerequisites

- [Node.js](https://nodejs.org) 20 or later
- [pnpm](https://pnpm.io) (the repo pins the version in `package.json`; with Corepack, run `corepack enable`)
- [Docker](https://www.docker.com) to run PostgreSQL (or any PostgreSQL instance you already have)
- A [GitHub OAuth App](https://github.com/settings/developers) for sign-in

### 1. Install dependencies

```bash
pnpm install
```

### 2. Start the database

```bash
docker compose up -d
```

This starts PostgreSQL 17 on `localhost:5432` with user `postgres`, password `postgres` and a database named `board`.

### 3. Configure environment variables

```bash
cp .env.example .env
```

| Variable | Description |
| --- | --- |
| `DATABASE_URL` | PostgreSQL connection string. The default matches `docker-compose.yml`. |
| `BETTER_AUTH_SECRET` | Random secret, at least 32 characters. Generate one with `openssl rand -base64 32`. |
| `BETTER_AUTH_URL` | Base URL of the app. Use `http://localhost:3000` locally. |
| `GITHUB_CLIENT_ID` | Client ID of your GitHub OAuth App. |
| `GITHUB_CLIENT_SECRET` | Client secret of your GitHub OAuth App. |
| `NEXT_PUBLIC_API_URL` | Optional. Base URL used by the client to reach the API. Defaults to `http://localhost:3000`. |

To get the GitHub credentials, create an OAuth App at **GitHub > Settings > Developer settings > OAuth Apps** with:

- **Homepage URL:** `http://localhost:3000`
- **Authorization callback URL:** `http://localhost:3000/api/auth/callback/github`

### 4. Set up the database schema

```bash
pnpm db:migrate
```

Optionally, fill the board with sample issues and comments (this truncates the existing issues and comments first):

```bash
pnpm db:seed
```

### 5. Run the app

```bash
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000).

## API documentation

With the app running, the interactive API reference is available at [http://localhost:3000/api/docs](http://localhost:3000/api/docs) and the raw OpenAPI spec at [http://localhost:3000/api/openapi.json](http://localhost:3000/api/openapi.json).

| Method | Path | Auth | Description |
| --- | --- | --- | --- |
| `GET` | `/api/issues` | No | List issues grouped by status, with optional search |
| `GET` | `/api/issues/interactions` | No | Like counts and the current user's like state for a set of issues |
| `GET` | `/api/issues/{id}` | No | Get an issue |
| `POST` | `/api/issues` | No | Create an issue |
| `PATCH` | `/api/issues/{id}` | No | Update an issue |
| `DELETE` | `/api/issues/{id}` | No | Delete an issue |
| `POST` | `/api/issues/{id}/like` | Yes | Toggle your like on an issue |
| `GET` | `/api/issues/{id}/comments` | No | List an issue's comments |
| `POST` | `/api/issues/{id}/comments` | Yes | Create a comment |
| `PATCH` | `/api/issues/{issueId}/comments/{commentId}` | Yes | Update a comment |
| `DELETE` | `/api/issues/{issueId}/comments/{commentId}` | Yes | Delete a comment |

Authentication endpoints are handled by Better Auth under `/api/auth/*`.

## Scripts

| Script | Description |
| --- | --- |
| `pnpm dev` | Start the development server |
| `pnpm build` | Create a production build |
| `pnpm start` | Run the production build |
| `pnpm lint` | Lint the code with ESLint |
| `pnpm db:generate` | Generate a migration from schema changes in `src/api/db/schema.ts` |
| `pnpm db:migrate` | Apply pending migrations |
| `pnpm db:push` | Push the schema directly to the database, without migration files |
| `pnpm db:studio` | Open Drizzle Studio to browse the data |
| `pnpm db:seed` | Reset and seed the database with sample data |

## Notes

- The Next.js version used here has breaking changes compared to older releases. When working on the code, check the guides in `node_modules/next/dist/docs/` (see `AGENTS.md`).
- The API's CORS configuration currently allows only `http://localhost:3000`; update `src/api/index.ts` if you deploy elsewhere.
