# Hopper Recon

A modern security reconnaissance platform combining a **Next.js 16 web frontend** with a **Go MCP engine** for OSINT and network discovery tasks.

## Architecture

```
hopper-recon/
├── web/                 # Next.js 16 frontend + SQLite local DB
├── engine/              # Go MCP server (runs in Docker)
├── CLAUDE.md            # Development guide
└── schema.sql           # Database schema (D1 production)
```

- **Web App** (`web/`): Next.js 16 with shadcn/ui, Tailwind CSS, and better-sqlite3 for local development
- **Engine** (`engine/`): Go MCP server wrapping OSINT tools (subfinder, dnsx, tlsx, httpx, asnmap)
- **Communication**: Docker container running the engine, called via MCP protocol from the web app

## Quick Start

### Prerequisites

- Node.js 18+
- Go 1.26+
- Docker (for running the Go engine)
- OSINT tool binaries: `subfinder`, `dnsx`, `tlsx`, `httpx`, `asnmap`

### Setup

1. **Clone and install**
   ```bash
   npm install
   cd engine && go mod download
   ```

2. **Build the Docker engine**
   ```bash
   cd engine && docker build -t hopper-recon:latest .
   ```

3. **Start development server**
   ```bash
   npm run dev
   ```
   Web app runs on `http://localhost:3000`

### Database

- **Development**: SQLite at `web/data/recon.db` (auto-created)
- **Production**: Cloudflare D1 (configured via Wrangler)
- **Schema**: Defined in `web/schema.sql` and `web/src/lib/db.ts`

## Available Tools

The Go engine exposes these MCP tools to the web app:

| Tool | Purpose | Input |
|------|---------|-------|
| **subfinder** | OSINT subdomain enumeration | Domain name |
| **dnsx** | DNS resolution & validation | Domain/subdomain |
| **tlsx** | TLS certificate & SAN extraction | Domain/IP |
| **httpx** | HTTP probing & tech detection | Domain/IP |
| **asnmap** | ASN & CIDR range mapping | Domain name |

## Development

### Before committing:

#### Next.js (web/)
```bash
npm run lint        # Fix ESLint warnings
npx tsc --noEmit   # Type check (no @ts-ignore)
npm run build       # Verify build
```

#### Go (engine/)
```bash
gofmt -w .          # Format all Go code
go vet ./...        # Check for errors
go build ./...      # Verify compilation
go mod tidy         # Update dependencies
docker build -t hopper-recon:latest .  # Rebuild container image
```

### Code style

- **No Prettier config**: Match surrounding file style (indentation, quotes, commas)
- **Type safety**: TypeScript strict mode; no `any` or `@ts-ignore`
- **Go formatting**: Must run `gofmt` before commit
- **Comments**: Only for non-obvious WHY, not for describing WHAT

## Project Structure

```
web/
├── src/
│   ├── app/           # Next.js 16 App Router
│   ├── components/    # React components (shadcn/ui)
│   ├── lib/
│   │   ├── db.ts      # SQLite schema & queries
│   │   ├── docker-mcp.ts  # MCP tool definitions
│   │   └── ...
│   └── styles/        # Tailwind + globals
├── data/              # SQLite DB (gitignored)
├── public/
├── package.json       # Dependencies
└── tsconfig.json

engine/
├── main.go            # MCP server & tool handlers
├── Dockerfile
├── go.mod
└── go.sum
```

## Making Changes

### Adding a new tool

1. **Implement in Go** (`engine/main.go`):
   - Define input/output types
   - Add handler function
   - Register with MCP server

2. **Update TypeScript** (`web/src/lib/docker-mcp.ts`):
   - Add to `McpTool` union type
   - Add to valid tools list in scan route

3. **Rebuild**:
   ```bash
   cd engine && docker build -t hopper-recon:latest .
   ```

### Database schema changes

- Edit `web/schema.sql` (D1 production target)
- Update inline schema in `web/src/lib/db.ts` (SQLite dev)
- Keep both in sync

## Deployment

### To Vercel (production)

```bash
npm run build
vercel deploy --prod
```

Environment variables configured via Vercel dashboard. Database uses Cloudflare D1.

## Useful Commands

```bash
# Web development
npm run dev              # Start dev server
npm run build            # Build for production
npm run lint             # Lint & format check
npx tsc --noEmit        # Type check

# Go/Engine
go doc <pkg>            # Read package docs
go doc <pkg>.<Symbol>   # Read specific API

# Docker
docker build -t hopper-recon:latest .    # Build image
docker run --rm -it hopper-recon:latest  # Test locally
```

## Resources

- [CLAUDE.md](./CLAUDE.md) — Detailed development guide & conventions
- [TODO.md](./TODO.md) — Outstanding work & issues
- Next.js 16 docs: `web/node_modules/next/dist/docs/`
- MCP SDK: `engine/vendor/github.com/modelcontextprotocol/go-sdk`

## License

Internal project