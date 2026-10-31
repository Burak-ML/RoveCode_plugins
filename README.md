# RoveCode Plugins

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![Runtime](https://img.shields.io/badge/runtime-Bun-black?logo=bun)
![Protocol](https://img.shields.io/badge/protocol-MCP-informational)

**A skill, routing and memory layer for AI coding tools, delivered as an MCP server.**

AI coding assistants either know nothing about your stack or load every instruction they have into the context window. RoveCode Plugins sits between your request and the model: it picks the few relevant skills, keeps a profile of your preferences, and returns a compact block of context. Routing is deterministic, so it needs no LLM call.

## Features

- **Deterministic routing.** A classifier maps each request to a domain, scans skill metadata without loading skill bodies, scores the candidates and enforces a token budget. Only the selected skills are loaded.
- **11 domain plugins, 72 skills.** Web design, backend, database, Rust, security, DevOps, testing, AI engineering, mobile, automation and game development.
- **Personal skill file.** One evolving file per user records likes, dislikes, "remember this" statements and verified fixes. Retrieval uses TF-IDF similarity with entity and recency scoring, and it is domain-aware, so a Rust preference never shows up in a web-design request.
- **Automatic learning.** After each turn the `analyze_and_learn` tool detects preference signals and updates the personal skill (add, merge, strengthen, weaken, replace or ignore).
- **Skill composition.** Skills declare related skills, so picking `creative-direction` also suggests `typography`, `layout-composition` and `color-system`.
- **Works with any MCP client.** Claude Code, OpenCode, VS Code (Kilo Code), Hermes Agent and anything that speaks stdio MCP.

## Quick start

Requires [Bun](https://bun.sh) 1.0 or later. No separate Node.js install is needed. Works on Windows, macOS and Linux.

```bash
git clone https://github.com/Burak-ML/RoveCode_plugins
cd RoveCode_plugins
bun install
bun test        # 149 tests should pass
```

Every client below needs the **absolute path** to `src/mcp/index.ts` in your clone. Forward slashes work on Windows too (`C:/Users/you/RoveCode_plugins/src/mcp/index.ts`).

### Claude Code

```bash
claude mcp add rovecode-plugins -- bun run /path/to/RoveCode_plugins/src/mcp/index.ts
claude mcp list    # should show: rovecode-plugins ... Connected
```

### OpenCode

Add to `~/.config/opencode/opencode.jsonc`:

```jsonc
{
  "mcp": {
    "rovecode-plugins": {
      "type": "local",
      "enabled": true,
      "command": ["bun", "run", "/path/to/RoveCode_plugins/src/mcp/index.ts"]
    }
  }
}
```

The `AGENTS.md` in this repository tells the model to call `route_request` and `get_context` before it answers.

### VS Code (Kilo Code) and other clients

Most clients accept the same stdio definition:

```json
{
  "mcpServers": {
    "rovecode-plugins": {
      "command": "bun",
      "args": ["run", "/path/to/RoveCode_plugins/src/mcp/index.ts"]
    }
  }
}
```

For Hermes Agent use the `mcp_servers` key with `"transport": "stdio"`.

## MCP tools

| Tool | Purpose |
|------|---------|
| `route_request` | Route a request and return the selected plugins, skills and personal entries |
| `get_context` | Assemble the full context, ready to inject into a system prompt |
| `record_learning` | Record an explicit preference or a verified solution |
| `analyze_and_learn` | Analyze a conversation turn and apply detected learning signals |

Check the server directly:

```bash
echo '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}' | bun run src/mcp/index.ts
```

## How routing works

```
"Build a responsive SaaS dashboard"
        |
  classifier           -> domain: web-design
        |
  plugin selection     -> web-design selected; rust, backend, game-development rejected
        |
  skill scoring        -> layout-composition, responsive-design, spacing-rhythm selected
        |                  typography, visual-review ... below the score threshold
  personal retrieval   -> "prefers narrow sidebar"
        |
  token budget         -> 2,650 of 6,000 tokens
        |
  compact context returned to the model
```

To see the decisions for any request:

```bash
bun run src/cli/index.ts route "Build a SaaS dashboard" default --debug
```

The CLI keeps a file watcher open so plugin edits hot-reload, so it does not return to the prompt by itself. Press `Ctrl+C` after the output.

## Plugin library

| Plugin | Skills |
|--------|--------|
| `web-design` | creative direction, typography, layout, responsive design, interaction, motion, accessibility, web performance, browser QA, visual review, spacing, color |
| `backend` | API design, authentication, middleware, HTTP error handling, rate limiting, Express patterns |
| `database` | schema design, query optimization, migrations, indexing, transactions, ORM patterns |
| `rust` | core, ownership and borrowing, async concurrency, performance, error handling, CLI engineering |
| `security` | auth security, input validation, dependency audit, XSS/CSRF, injection prevention, secrets hygiene |
| `devops` | Docker, CI/CD, environment config, logging and monitoring, deployment strategies, secrets management |
| `testing` | unit, integration and end-to-end testing, TDD, mocking, performance testing |
| `ai-engineering` | prompt engineering, RAG, fine-tuning, agent design, embeddings, LLM evaluation |
| `mobile` | React Native, mobile performance, navigation, offline storage, push notifications, app store deployment |
| `automation` | workflow design, browser automation, scheduling, data transformation, scraping, retries |
| `game-development` | game loop, physics, rendering pipeline, input handling, game state, asset management |

## Benchmarks

The benchmarks in this repository are written and run by the project, so treat them as internal evaluation, not independent measurements.

| Suite | Scope |
|-------|-------|
| `benchmark/run-benchmark.ts` | 20 hand-picked scenarios across 9 domains (routing accuracy, token use, speed, plugin isolation) |
| [`benchmark/large/`](benchmark/large/) | 445 scenarios derived from every skill's activation data, plus `bun run lint:signals` to catch signal collisions between skills |
| [`benchmark/learning/`](benchmark/learning/) | 57 scenarios for signal detection and domain isolation in the personal-learning system |

```bash
bun run benchmark/run-benchmark.ts
bun run benchmark:large
bun run benchmark:learning
```

## Troubleshooting

- **Client cannot start the server.** Run the exact `bun run /path/.../src/mcp/index.ts` command from your config in a terminal. A wrong path or a `bun` that is not on the client's `PATH` is the usual cause, and the terminal shows the real error.
- **Learned preferences disappeared.** Personal data lives in `data/users/` inside your clone and is git-ignored on purpose. Deleting or moving the clone deletes that history.

## License

MIT. See [LICENSE](LICENSE).
