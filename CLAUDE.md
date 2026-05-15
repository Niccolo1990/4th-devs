# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is **ai-devs-4**, a comprehensive curriculum of 24 lessons teaching AI-powered application development with Claude. It contains ~60 example projects spanning:
- Basic API interactions (Lesson 01)
- Tool use and Model Context Protocol (Lessons 02-03)
- Multimodal capabilities (Lesson 04)
- Agent orchestration and databases (Lessons 05-10)
- Observability and evals (Lesson 11)
- Multi-agent systems and integrations (Lessons 12-14)
- Advanced UIs and real-time features (Lessons 15-22)
- Prompt optimization and production APIs (Lessons 23-24)

Each lesson is organized in numbered directories (01_01_*, 02_02_*, etc.) with self-contained examples that can run independently.

## Setup & Requirements

- **Node.js 24+** (required, enforced by config.js)
- **Environment**: Copy `env.example` to `.env` at repo root for shared API keys
- **Package managers**: Mix of npm, bun, and pnpm across projects
- **API Keys**: Set either `OPENAI_API_KEY` or `OPENROUTER_API_KEY` (defaults to OpenAI if both present)

Key env vars (see README.md for full list):
- `OPENAI_API_KEY` or `OPENROUTER_API_KEY` — LLM provider
- `GEMINI_API_KEY` — Google Gemini for media/images
- `REPLICATE_API_TOKEN` — Image/video generation
- `NEO4J_URI`, `NEO4J_USERNAME`, `NEO4J_PASSWORD` — Graph agents (Lesson 08)
- `LANGFUSE_*` — Observability (Lesson 11)
- `LIVEKIT_URL`, `LIVEKIT_API_KEY`, `LIVEKIT_API_SECRET` — Voice agents (Lesson 22)

## Project Structure

```
├── 01_XX_* — Lessons 1-5: Foundations (basic API, tools, MCP, multimodal, agents)
├── 02_XX_* — Lessons 6-10: RAG, embeddings, graphs, ops, sandbox
├── 03_XX_* — Lessons 11-15: Observability, multi-agent, browser, Gmail, advanced UIs
├── 04_XX_* — Lessons 16, 19-20: Garden, system ops, marketing apps, review tool
├── 05_XX_* — Lessons 21-24: Agent graph, voice/chat UI, prompt opt, production API
├── mcp/ — MCP server implementations (files-mcp, uploadthing-mcp, web)
├── config.js — Shared provider/env resolution (AI_PROVIDER, API_KEY, model resolution)
├── package.json — Root scripts for all lessons (lesson1:install, lesson5:agent, etc.)
└── ensure-files-mcp.mjs — Helper to ensure files-mcp is built before certain examples
```

## Common Development Commands

**Install all dependencies for a lesson:**
```bash
npm run lesson1:install   # Lesson 1 examples
npm run lesson5:install   # Lesson 5 examples
npm run lesson24:install  # Lesson 24 (API server)
```

**Run an example:**
```bash
npm run lesson1:interaction
npm run lesson5:agent
npm run lesson24:api      # Starts server
npm run lesson24:ui       # Starts UI (separate terminal)
```

**Lessons with setup steps:**
- **Lesson 05 agent** (01_05_agent): Database required before first run:
  ```bash
  npm run lesson5:agent:db:push
  npm run lesson5:agent:db:seed
  ```
- **Lesson 24 API** (05_04_api): Database migration + seed:
  ```bash
  npm run lesson24:api:db:migrate
  npm run lesson24:api:db:seed
  ```
- **Lesson 08 graph agents** (02_03_graph_agents): Requires running Neo4j 5.11+:
  ```bash
  docker run -d --name neo4j -p 7474:7474 -p 7687:7687 -e NEO4J_AUTH=neo4j/password neo4j:5
  ```
- **Lesson 13 browser** (03_03_browser): Login once to Goodreads:
  ```bash
  npm run lesson13:browser:login
  ```
- **Lesson 14 Gmail** (03_04_gmail): OAuth setup required:
  ```bash
  npm run lesson14:gmail:auth
  ```

**Format & lint:**
- **05_04_api** (Lesson 24) uses Biome:
  ```bash
  cd 05_04_api && npm run lint && npm run lint:fix
  npm run format && npm run format:fix
  npm run typecheck
  ```

**Development watch mode:**
- Most TypeScript projects support `npm run dev` or `bun run dev`
- Some use `tsx watch` internally
- Lesson 22 UI runs server + UI concurrently: `npm run dev` (opens two terminals)

## Architecture & Patterns

### Configuration & Provider Resolution
- **config.js** (root): Central resolver for AI providers, API keys, and model names
  - Validates Node.js version
  - Loads `.env` files (root + per-project)
  - Exports: `AI_PROVIDER`, `AI_API_KEY`, `RESPONSES_API_ENDPOINT`, `CHAT_API_BASE_URL`
  - Helper: `resolveModelForProvider(model)` — normalizes model names for OpenRouter (e.g., `gpt-4` → `openai/gpt-4`)
  - Helper: `buildResponsesRequest()` — constructs request payloads with web search, tools, plugins
- Per-project `package.json` files may define custom `.env` paths; those take priority over root

### API Integration Patterns
- **Responses API**: Core request/response pattern via OpenAI/OpenRouter HTTP endpoints
  - Used in Lessons 01-04 for basic requests, function calling, tool use
  - Request shape: `{ model, input: [], tools?, plugins?, webSearch?, ... }`
  - Response: `{ output: { type, text }, usage, error?, ... }`

- **OpenAI SDK**: Used in agent/production examples (Lessons 05+, 11, 14, 24)
  - Direct `new Anthropic()` and `new OpenAI()` initialization
  - Some examples use `@anthropic-ai/sdk` for Claude API

- **Model Context Protocol (MCP)**: Core abstraction for tool/resource access
  - Server spawned via stdio transport or HTTP
  - Examples: `files-mcp`, `uploadthing-mcp` (in `/mcp` folder), custom MCP servers per lesson
  - Client libraries: `@modelcontextprotocol/sdk` (TypeScript), MCP servers defined in `mcp.json` configs

### Database & Persistence
- **Drizzle ORM**: Primary choice for schema management and migrations
  - Used in: Lesson 05 agent (01_05_agent), Lesson 24 API (05_04_api)
  - Config: `drizzle.config.ts` + `src/db/schema.ts`
  - Migrations: `npm run db:push` (dev) or `npm run db:migrate` (prod)
  - Seed scripts: `src/db/seed.ts` or `src/db/seed-main-account.ts`

- **SQLite**: Local development database (better-sqlite3 for Lesson 24)
  - Lesson 07 hybrid RAG: FTS5 full-text search + sqlite-vec vector indexes

- **Neo4j**: Graph database (Lesson 08)
  - Used for knowledge graph + RAG agent
  - Requires 5.11+ for vector index support

### Agent & Multi-Agent Patterns
- **Simple loops** (Lessons 02, 03): Request → tool call → append result → repeat
- **Agentic RAG** (Lesson 06): Retrieval loop with conversation history
- **Context engineering** (Lesson 10): Observer/reflector pattern for memory management
- **Multi-agent systems** (Lessons 12, 19, 21): Task delegation, phase sequencing, heartbeat loops
- **Live UIs** (Lessons 15, 20, 22): Real-time streaming via WebSocket/SSE, artifact preview
- **Observability** (Lesson 11): Langfuse tracing at adapter boundary for cost/quality insights
- **Prompt optimization** (Lesson 23): LLM judge + hill-climbing candidate generation

### UI & Frontend
- **Svelte 5**: Primary framework for complex UIs (Lessons 15, 20, 22, 24)
  - Vite dev server, Tailwind CSS, custom component patterns
  - Live preview/artifact rendering via WebSocket sync
  - TipTap editor integration (Lesson 24)

- **Simple HTML + JavaScript**: Used in simpler examples (Lesson 01)
- **Server-side rendering**: Bun native HTTP + fetch-based APIs

### Tooling & Runtimes
- **Node.js**: Primary (most examples)
- **Bun**: Increasingly used for speed (newer lessons 03_02_*, 03_03_*, 03_05_*, 04_05_*, 05_02_*, 05_03_ax)
  - Bun also provides TypeScript transpilation without explicit `tsx`
  - Some projects use `bun.lock` instead of `package-lock.json`

- **Deno**: Code execution sandbox (Lesson 12 code example, 03_02_code)
- **QuickJS**: JavaScript sandbox (Lesson 10 sandbox agent)

### Key Shared Modules
- **config.js**: Provider resolution, env loading, request building
- **helpers.js**: Common utilities (message formatting, response extraction)
- **MCP server (mcp/files-mcp)**: File operations wrapped as MCP tools
- **Langfuse integration**: Tracing boilerplate (Lessons 11, 24)

## Notable Lesson Patterns

| Lesson | Focus | Key Tech | Notes |
|--------|-------|----------|-------|
| 01 | Basic API, JSON output | Responses API | Introduces model interaction |
| 02-03 | Tool use, MCP | Function calling, stdio transport | Foundation for agent patterns |
| 04 | Multimodal (audio, video, images) | Gemini, Replicate | Heavy use of media APIs |
| 05 | Agent orchestration | Hono, Drizzle, MCP | First full-stack example |
| 06-10 | RAG, graphs, multi-agent | SQLite, Neo4j, memory patterns | Complex agent loops |
| 11 | Observability | Langfuse, OpenTelemetry | Cost tracking at adapter boundary |
| 12-14 | Multi-agent, integrations | Deno, Gmail OAuth, browser automation | Real-world integrations |
| 15 | Advanced UIs | Svelte, WebSocket | Live rendering, artifacts |
| 16 | Digital garden | Markdown workspace | Personal knowledge base |
| 19-20 | System-level agents | MCP dashboards, document review | Complex workflows |
| 21-22 | Graph orchestration, voice | Cytoscape, LiveKit, realtime | Advanced real-time features |
| 23 | Prompt optimization | DSPy/Ax, few-shot learning | Training loop for prompts |
| 24 | Production API server | Hono + Drizzle + Svelte | Full-stack with multi-tenant DB |

## Debugging & Development Tips

1. **Provider mismatch errors**: Check `config.js` — ensure both OpenAI/OpenRouter keys are set if using OpenRouter
2. **Model name resolution**: OpenRouter requires `openai/gpt-4` syntax; config.js handles this automatically
3. **MCP server issues**: Check `mcp.json` config and ensure the server binary is built (`npm run ensure:files-mcp` pre-hook)
4. **Database schema mismatches**: Run `npm run db:push` or `npm run db:generate` + `npm run db:migrate`
5. **Missing workspace files**: Some examples expect files in `workspace/` — copy from example data or use `DEMO_DATASET_FILE` env var
6. **Port conflicts**: API servers default to 3000; override with env vars or `package.json` scripts
7. **Environment isolation**: Each `03_05_*` example has its own `.env` — per-project keys take priority

## Key Files to Know

- **config.js**: Provider/model resolution, env loading
- **package.json** (root): All lesson npm scripts
- **README.md**: Full setup instructions, lesson-by-lesson CLI commands
- **mcp/files-mcp/**: Built MCP server for file operations
- **01_05_agent/, 05_04_api/**: Reference full-stack architectures with Drizzle + Hono
- **03_02_events/, 04_04_system/**: Multi-agent orchestration examples
- **03_05_*, 04_05_apps/**: Advanced UI + agent integration patterns

