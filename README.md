# Forge

> **An AST visualizer and Go code prototyping tool** that lets developers explore Go file structure as interactive node graphs and edit AST nodes through an intuitive sidebar UI.

Forge parses Go source files and renders their Abstract Syntax Tree (AST) as a zoomable, interactive flow canvas — making it easy to understand code structure, trace symbol usages, and prototype new Go code directly in the browser.

---

## Features

- 📂 **Project Import** — Upload a `.zip` archive or clone directly from a GitHub/GitLab URL
- 🌳 **AST Visualisation** — Explore your Go file's full AST as an interactive node graph powered by React Flow
- ✏️ **In-canvas Editing** — Modify AST node properties from a sidebar panel; edited nodes are flagged with an amber indicator
- 🧩 **Node Palette** — Drag-and-drop Go AST node templates onto the canvas to prototype new structures
- 🔍 **Inspector Panel** — Deep-dive into node metadata and symbol usages across your project
- 🗂️ **Multi-tab Canvas** — Open multiple files simultaneously in separate tabs
- 🔐 **Auth** — JWT-based register / login with email-based password reset flow

---

## Architecture

Forge is a monorepo with two services:

```
Forge/
├── forge-frontend/   # React + TypeScript SPA
└── forge-backend/    # Go REST API
```

### Frontend (`forge-frontend`)

| Technology | Role |
|---|---|
| React 18 + TypeScript | UI framework (strict mode) |
| Vite | Build tool & dev server |
| Tailwind CSS | Utility-first styling |
| @xyflow/react (React Flow v12) | Interactive AST node graph canvas |
| Zustand | Global state — multi-tab `TabState` pattern |
| TanStack Query v5 | Server state & API mutations |
| Axios | HTTP client |
| dagre | Automatic hierarchical graph layout |
| Radix UI | Accessible headless components |
| Monaco Editor | In-browser code editing |
| lucide-react | Icon library |
| react-hot-toast | Toast notifications |

### Backend (`forge-backend`)

| Technology | Role |
|---|---|
| Go | Language |
| Fiber v2 | HTTP framework |
| PostgreSQL + sqlx | Persistence |
| go-git | Git clone for URL import |
| JWT (golang-jwt/jwt v5) | Authentication tokens |
| bcrypt | Password hashing |
| Zap | Structured logging |
| Viper | Config / env management |

---

## Getting Started

### Prerequisites

- **Go** ≥ 1.21
- **Node.js** ≥ 18
- **PostgreSQL** running locally (or a connection string)

---

### 1. Clone the repo

```bash
git clone https://github.com/Drexterr/Forge.git
cd Forge
```

---

### 2. Backend setup

```bash
cd forge-backend

# Copy environment variables
cp .env.example .env
# Edit .env with your database URL, JWT secret, etc.

# Run the server
go run ./cmd/main.go
```

The API will start on **http://localhost:8080**.

**Environment variables (`forge-backend/.env`):**

| Variable | Description |
|---|---|
| `SERVER_PORT` | Port the API listens on (default `8080`) |
| `DATABASE_URL` | PostgreSQL connection string |
| `JWT_SECRET` | Minimum 32-character secret for signing JWTs |
| `AI_API_KEY` | AI provider API key |
| `ENVIRONMENT` | `dev` or `production` |
| `SMTP_HOST` | SMTP server host |
| `SMTP_PORT` | SMTP server port |
| `SMTP_USER` | SMTP username / email |
| `SMTP_PASSWORD` | SMTP password / app password |
| `SMTP_FROM` | Sender address for system emails |

---

### 3. Database

Apply the schema migrations found in `forge-backend/db/` to your PostgreSQL instance before starting the server.

---

### 4. Frontend setup

```bash
cd forge-frontend

# Copy environment variables
cp .env.example .env.local
# Edit .env.local — set VITE_API_URL and VITE_WS_URL to your running backend

# Install dependencies
npm install

# Start dev server (port 3000)
npm run dev
```

Open **http://localhost:3000** in your browser.

**Environment variables (`forge-frontend/.env.local`):**

| Variable | Description |
|---|---|
| `VITE_API_URL` | Base URL of the backend API (e.g. `http://localhost:8080`) |
| `VITE_WS_URL` | WebSocket URL of the backend |

API calls are proxied through Vite — see `vite.config.ts` for proxy configuration.

---

## API Reference

### Auth

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/auth/register` | Create a new account |
| `POST` | `/api/auth/login` | Log in and receive a JWT |
| `GET` | `/api/auth/me` | Get the current authenticated user |
| `POST` | `/api/auth/forgot-password` | Request a password reset token |
| `POST` | `/api/auth/reset-password` | Reset password using a token |

### AST

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/ast/inspect` | Upload a `.go` file → receive AST summary |
| `POST` | `/api/ast/inspect-raw` | Send raw Go source → receive AST summary |
| `POST` | `/api/ast/tree` | Upload a `.go` file → receive full recursive AST tree |
| `POST` | `/api/ast/tree-raw` | Send raw Go source → receive full recursive AST tree |

### Projects

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/projects` | List all projects for the authenticated user |
| `POST` | `/api/projects/upload` | Import a project from a `.zip` file (≤ 100 MB) |
| `POST` | `/api/projects/import` | Import a project from a GitHub/GitLab URL |
| `GET` | `/api/projects/:id/files` | List files in a project |
| `GET` | `/api/projects/:id/files/*` | Get a specific file's content |
| `GET` | `/api/projects/:id/usages` | Get symbol usage analysis |

---

## Frontend Directory Structure

```
forge-frontend/src/
├── components/
│   ├── ast/          # AST canvas, sidebar, node palette, individual node card
│   ├── dashboard/    # Dashboard components
│   ├── inspector/    # AST Inspector panel
│   ├── tree/         # React Flow tree visualizer nodes and canvas
│   ├── ui/           # Base UI primitives (Radix-based)
│   └── upload/       # Upload modal and processing components
├── layouts/          # App shell layouts
├── lib/              # Utilities — api.ts, astToGraph.ts, dagreLayout.ts
├── pages/            # Page components (ASTPage.tsx is the main canvas page)
├── store/            # Zustand stores — astViewerStore.ts (core state)
└── styles/           # Global CSS
```

---

## Frontend Commands

```bash
npm run dev       # Start dev server on port 3000
npm run build     # Type-check + production build (output: dist/)
npm run preview   # Preview production build locally
npm run lint      # ESLint check
npx tsc --noEmit  # TypeScript type check (must be zero errors)
```

---

## Backend Directory Structure

```
forge-backend/
├── api/
│   ├── handlers/     # HTTP handler implementations
│   ├── middleware/   # JWT auth middleware
│   └── router/       # Route registration
├── cmd/
│   └── main.go       # Entry point
├── db/               # SQL migrations
└── internal/
    ├── ast/          # Go AST parsing logic
    ├── auth/         # User model, repository, JWT helpers
    ├── config/       # Environment config loading
    ├── db/           # Database connection helpers
    ├── logger/       # Zap logger factory
    ├── project/      # Project & file repository
    └── upload/       # ZIP extraction and git clone logic
```

---

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feat/your-feature`)
3. Commit your changes
4. Push and open a Pull Request

---

## License

MIT
