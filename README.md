# 🤖 Jaisiu

> **Any OS gateway for AI agents across WhatsApp, Telegram, Discord, iMessage, and more.**
> Send a message, get an agent response from your pocket. Plugins add Mattermost and more.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge)](LICENSE)

**Jaisiu** is the multi-channel runtime that turns a local AI model into a real assistant you can talk to from any chat app, paired phone, or the bundled web UI. It also ships the enterprise file portal (EFSS), optional TRELLIS image→3D, and a self-improvement loop for harness work.

## Key Features

- **Employee Management** — Manage AI agents as employees with full HR metadata (name, role, department, status, etc.)
- **HR System Synchronization** — Pluggable connector system for syncing employee data with external HR systems
- **Multi-Channel Presence** — Virtual employees can operate across WhatsApp, Telegram, Slack, Discord, Google Chat, Signal, iMessage, Microsoft Teams, Matrix, and more
- **Channel Access Control** — Fine-grained permissions for which channels each virtual employee can access
- **Admin Panel** — Web-based control UI for managing employee agents, their channel access, and permissions
- **Voice & Canvas** — Voice Wake, Talk Mode, and live Canvas support for interactive experiences
- **EFSS + 3D generation** — Enterprise file portal with optional TRELLIS image-to-GLB inference via the EFSS viewer

## Employee Management System

Jaisiu introduces a comprehensive Employee Management & HR Sync System:

### Core Features

- **Employee Metadata** — Each agent can have employee information (ID, name, email, department, role, status)
- **Channel Permissions** — Control which channels each employee agent can access with granular permissions
- **HR Connectors** — Pluggable system supporting mock, file-based, and custom HR system connectors
- **Auto-Sync** — Automatically create/update/deactivate agents based on HR system data
- **Admin UI** — Manage employees and HR sync from the Control Panel

### Configuration Example

```json5
{
  employees: {
    enabled: true,
    sync: {
      connector: "mock",
      connectors: {
        mock: {
          type: "mock",
          employees: [
            {
              employeeId: "EMP001",
              name: "Virtual Sales Rep",
              email: "sales-bot@company.com",
              department: "Sales",
              role: "Sales Representative",
              status: "active",
            },
          ],
        },
      },
      schedule: "0 2 * * *",
      autoCreateAgents: true,
      autoDeactivateAgents: true,
      defaultChannels: ["slack", "teams"],
    },
  },
  agents: {
    list: [
      {
        id: "sales-agent",
        name: "Sales Assistant",
        employee: {
          employeeId: "EMP001",
          name: "Virtual Sales Rep",
          email: "sales-bot@company.com",
          department: "Sales",
          role: "Sales Representative",
          status: "active",
          channels: {
            allowed: ["slack", "teams", "whatsapp"],
            permissions: {
              slack: { permission: "full", teams: ["sales-team"] },
              teams: { permission: "full" },
              whatsapp: { permission: "read-only" },
            },
          },
        },
      },
    ],
  },
}
```

## For developers (repo access)

You have GitLab access and need a working local harness **today**. Do this.

### Prerequisites

- **Node ≥ 22**, **pnpm** (`corepack enable && corepack prepare pnpm@10.23.0 --activate`)
- **Docker** + Compose v2 (`docker compose version`)
- Git identity for this clone (required by `git-hooks/commit-msg`):

```bash
git config --local user.email agents@pryzm.at
git config --local user.name "Jaisiu Commander"
```

### One-command first boot

```bash
git clone https://test.pryzmat.pl/root/jaisiu.git
cd jaisiu
./scripts/developer-setup.sh          # pnpm install, image build, compose up, verify
./scripts/verify-jaisiu-stack.sh
```

Then open **http://localhost:18789/** (Control UI / WebChat).

| Service              | URL                                                    |
| -------------------- | ------------------------------------------------------ |
| Gateway / Control UI | http://localhost:18789/                                |
| EFSS portal          | http://localhost:8090/efss/                            |
| Local LLM broker     | http://127.0.0.1:18800/v1 (`pryzm-llm-broker`)         |
| TRELLIS viewer (opt) | http://localhost:8090/efss/trellis/trellis-viewer.html |

### Pick how you talk to models (`JAISIU_DEV_LLM`)

**Customer / production** always defaults to **`pryzm-at-broker/MiniMax-M3`** (`config/config.yaml`) so installs work and models can switch through the broker.

**Developers** must not fight that default with ad-hoc edits. Switch packaging instead:

| Mode                         | Command                                        | Primary traffic                            |
| ---------------------------- | ---------------------------------------------- | ------------------------------------------ |
| **BYO MiniMax** (token plan) | `JAISIU_DEV_LLM=byo make up-dev-llm`           | `minimax-portal` — needs `MINIMAX_API_KEY` |
| **Local broker**             | `JAISIU_DEV_LLM=local-broker make up-dev-llm`  | compose broker `:18800`                    |
| **Remote broker**            | `JAISIU_DEV_LLM=remote-broker make up-dev-llm` | `https://api.pryzm.at/v1`                  |
| **BYO OpenAI**               | `JAISIU_DEV_LLM=byo-openai make up-dev-llm`    | `openai/gpt-5.2` — needs `OPENAI_API_KEY`  |

Profiles live in [`config/profiles/`](config/profiles/README.md). The gateway also honors `JAISIU_DEV_LLM` via YAML merge (no extra compose file required). Explicit `JAISIU_CONFIG_YAML_MERGE` still wins last.

```bash
# Typical PRIZM engineer box (MiniMax token plan):
export MINIMAX_API_KEY='sk-…'
JAISIU_DEV_LLM=byo make up-dev-llm

# Prove packaging invariants (also a GitLab hard gate before install-publish):
make verify-llm-packaging
```

**Never** set `JAISIU_DEV_LLM` on customer/production compose (`docker-compose.customer.yml`).

### Developer license (`developer-lifetime`)

Developers get an **open instance**: unified EdDSA JWT, unmetered broker auth, no harness concurrency overlay. License does **not** choose your model primary — packaging does.

```bash
# Operator mint (needs signing key access — ask a maintainer if you lack it):
node scripts/license/mint-license.mjs \
  --tier=developer-lifetime \
  --email=you@pryzm.at

# Put the JWT in .env (do not commit .env):
#   PRYZM_NODE_KEY=eyJ…
# Optional: same JWT as broker Bearer (or leave PRYZM_BROKER_KEY unset —
# env substitution can fall back to PRYZM_NODE_KEY).
```

Retired prefixes (`pzk_`, `pryzm_cu_`, `yo-momma`) are **refused**. Canonical licensing doc: [docs/efss/pryzm-broker-integration.md](docs/efss/pryzm-broker-integration.md).

### Daily loop

```bash
pnpm install
pnpm jaisiu:rebuild:fast          # refresh dist/
make build-all                    # refresh local images when Docker-backed
JAISIU_DEV_LLM=byo make up-dev-llm
pnpm check && pnpm test           # before you push
make verify-llm-packaging
```

Branch off `develop` for non-trivial work. Commit as **Jaisiu Commander \<agents@pryzm.at\>** (hook-enforced). Full contributor notes: [CONTRIBUTING.md](CONTRIBUTING.md) · [AGENTS.md](AGENTS.md) · [docs/book/13-development-and-testing.md](docs/book/13-development-and-testing.md).

### Customer CLI install (not the engineer path)

Runtime: **Node ≥22**. End-users / production nodes:

```bash
npm install -g jaisiu@latest
jaisiu onboard --install-daemon
# or: curl -fsSL https://pryzm.at/efss/jaisiu/install.sh | bash
```

Quick agent smoke (after gateway is up):

```bash
jaisiu gateway --port 18789 --verbose
jaisiu agent --message "Ship checklist" --thinking high
```

## Docker deploy (gateway + EFSS)

Full stack on one machine (same as [For developers](#for-developers-repo-access)):

```bash
./scripts/developer-setup.sh
# or pick an LLM mode: JAISIU_DEV_LLM=byo make up-dev-llm
./scripts/verify-jaisiu-stack.sh   # all checks must pass
```

**Runbooks:** [Complete deploy guide](docs/deploy/COMPLETE-DEPLOY-GUIDE.md) · [Quick reference](docs/deploy/QUICK-REFERENCE.md) · [EFSS integration](docs/efss/jaisiu-integration.md) · [Developer LLM profiles](config/profiles/README.md)

### Windows manual download

The native Windows installer (`install-native.ps1`) pulls a ~hundred-MB
tarball from `https://pryzm.at/efss/jaisiu/`. On a healthy artifact that
takes a few seconds, but a flaky ISP, a corporate TLS proxy, or a
real-time AV scan can stall or corrupt the download — even when the
artifact itself is fine. Don't wait for a pipeline: pre-stage the
tarball yourself and point the installer at the local copy.

```powershell
# 1. Install (PowerShell) — try this first
iwr -useb https://pryzm.at/efss/jaisiu/install-native.ps1 | iex

# 2. If the installer prints "Download FAILED after 3 attempts",
#    pre-stage the tarball and re-run with $env:JAISIU_BINARY:
iwr -useb https://pryzm.at/efss/jaisiu/jaisiu-gateway-windows-x64.tar.gz `
     -OutFile $env:USERPROFILE\Downloads\jaisiu.tar
$env:JAISIU_BINARY = "$env:USERPROFILE\Downloads\jaisiu.tar"
iwr -useb https://pryzm.at/efss/jaisiu/install-native.ps1 | iex
```

The installer skips its own download when `$env:JAISIU_BINARY` points
to a readable file (sha256 + size are verified against the release
manifest), so this works even when the network is broken. If the
download itself still fails, see the [download-failure
troubleshooting](docs/install/installer.md#download-failed-windows)
entry — it covers proxy workarounds, AV exclusions, and when to fall
back to WSL2.

### Troubleshooting

```bash
# If ./scripts/developer-setup.sh is not found, use the restart script:
./scripts/jaisiu/restart-jaisiu-docker.sh

# If permission errors occur:
chmod -R 777 .

# If after chmod you cannot pull due to permission-only changes,
# tell Git to ignore file permission changes:
git config core.fileMode false
```

## Channels

Jaisiu supports multiple communication channels for your virtual employees:

- **WhatsApp** — via Baileys
- **Telegram** — via grammY
- **Slack** — via Bolt
- **Discord** — via discord.js
- **Google Chat** — via Chat API
- **Signal** — via signal-cli
- **iMessage** — via imsg (macOS only)
- **Microsoft Teams** — via Bot Framework (extension)
- **Matrix** — via extension
- **WebChat** — built-in

### Rebuilding Jaisiu

When you change gateway, EFSS, CLI, or shared code, rebuild in this order — the harness image depends on the pnpm build, and docker compose reads the image:

```bash
# 1. Fast pnpm rebuild of all workspace packages (gateway, EFSS, CLI, broker, …)
pnpm jaisiu:rebuild:fast

# 2. Build every local Docker image defined in docker-compose.yml
make build-all

# 3. Recreate containers with the freshly built tags
docker compose up -d --force-recreate
```

Skipping any of the three leaves you running stale code: a `pnpm`-only rebuild doesn't refresh the image, and `make build-all` without `pnpm jaisiu:rebuild:fast` bakes an image with old `dist/`.

After containers are back up, hard-reload the browser tab pointing at the gateway WebChat / Control UI to drop any cached JS/CSS from the previous build:

- macOS / Windows / Linux: <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>R</kbd> (Firefox/Chrome/Edge; on macOS also <kbd>⌘</kbd>+<kbd>Shift</kbd>+<kbd>R</kbd>)

## Architecture

```
WhatsApp / Telegram / Slack / Discord / Google Chat / Signal / iMessage / Microsoft Teams / Matrix / WebChat
               │
               ▼
┌───────────────────────────────────────┐
│              Gateway                  │
│         (control plane)               │
│       ws://127.0.0.1:18789            │
│                                       │
│  ┌─────────────────────────────────┐  │
│  │    Employee Management System   │  │
│  │  - HR Sync Connectors           │  │
│  │  - Channel Access Control       │  │
│  │  - Employee Admin Panel         │  │
│  └─────────────────────────────────┘  │
└──────────────┬────────────────────────┘
               │
               ├─ Agent Runtime (RPC)
               ├─ CLI (jaisiu …)
               ├─ WebChat UI
               ├─ macOS app
               └─ iOS / Android nodes
```

## Key Subsystems

- **Gateway WebSocket Network** — Single WS control plane for clients, tools, and events
- **Employee Management** — HR-like management of AI agents with sync capabilities
- **Browser Control** — Managed Chrome/Chromium with CDP control
- **Canvas + A2UI** — Agent-driven visual workspace
- **Voice Wake + Talk Mode** — Always-on speech for macOS/iOS/Android
- **Nodes** — Canvas, camera, screen record, location, notifications
- **Prism / MemPrism / Shared Memory** — Redis-backed cross-agent state and recall
- **Floci** — Crowd orchestrator for multiple agent harnesses
- **TRELLIS.2** — Neural image-to-3D inference (`trellis_inference/`, port **18800**, GLB output via EFSS viewer)
- **trellis-3d** — Separate procedural WebGL primitive service (`src/trellis-3d/`, compose profile `with-trellis-3d`, port **18793**); not the TRELLIS.2 model pipeline
- **Self-Improvement / Self-Upgrade Loop** — Fleet self-improvement (single goal, YouTrack → code → push) extended into a self-upgrading harness loop: benchmark → propose → execute → score → repeat. See [JAISIU-196](https://pryzmat.youtrack.cloud/issues/JAISIU-196) Phase 5.
- **JAISIU-Bench** — Three-tier benchmark suite (dev set, held-out test set, context retention suite) for measuring harness quality. See [JAISIU-200](https://pryzmat.youtrack.cloud/issues/JAISIU-200).
- **Quantum Palace Reasoning Engine** — Palace as reasoning substrate (not just retrieval): multi-dimensional activation, activity graph as episodic memory, context retention over long sessions. Phase 2 (deferred). See `docs/tickets/JAISIU-BENCH.md`.

## Agent context & memory (Jaisiu)

Cross-agent recall and prompt budget are governed by:

- **Read cache / tool output policy** — `src/agents/` (`read-session-cache`, `tool-output-policy`, `prompt-budget`)
- **MemPrism** — optional `project` tag on writes for scoped recall (`src/memprism/`)
- **Skills gitignore** — agent skill loading filters (`src/agents/skills/`)

Umbrella planning ticket: [docs/tickets/JAISIU-CONTEXT-FABRIC-MASTER.md](docs/tickets/JAISIU-CONTEXT-FABRIC-MASTER.md)

## Documentation (in-repo)

| Doc                                                                      | Description                                                                                      |
| ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------ |
| **[docs/book/README.md](docs/book/README.md)**                           | **The Jaisiu Book** — readable guide from intro through deployment, EFSS, Prism, security        |
| [ARCHITECTURE.md](ARCHITECTURE.md)                                       | **Software structure:** hexagonal layout (EFSS, shared-memory), directory→layer map, target      |
| [docs/README.md](docs/README.md)                                         | **Documentation index:** how docs are organized (start, concepts, gateway, CLI, architecture)    |
| [docs/architecture/README.md](docs/architecture/README.md)               | **Architecture hub:** links to all architecture and diagram docs                                 |
| [docs/class-diagrams.md](docs/class-diagrams.md)                         | Mermaid class diagrams: PiEmbeddedRunner, SessionManager, MemPrism, SharedMemory, EFSS hexagonal |
| [docs/timestep-diagrams.md](docs/timestep-diagrams.md)                   | Sequence diagrams: LLM call flow, tool calls, gateway connection lifecycle                       |
| [docs/architecture-diagrams.md](docs/architecture-diagrams.md)           | High-level system and agent orchestration architecture                                           |
| [docs/concepts/architecture.md](docs/concepts/architecture.md)           | Gateway architecture, WS protocol, nodes, pairing                                                |
| [efss-portal/README.md](efss-portal/README.md)                           | **EFSS web portal** (pryzm.at): static browser, `pryzm-theme.css`, viewer, Redis doc publish     |
| [docs/trellis-inference-pipeline.md](docs/trellis-inference-pipeline.md) | **Image→3D:** TRELLIS API, Docker/ROCm runtime, GLB output, viewer operations                    |
| [docs/scripts.md](docs/scripts.md)                                       | **Scripts index:** `scripts/jaisiu/`, `scripts/efss/`, helpers, deploy entry points              |

**Critical code flow (message → agent → response):** Entry → `cli/run-main.ts` → gateway subcommand → `gateway/server.impl.ts` (`startGatewayServer`) → WS message handler → `gateway/server-methods.ts` (`handleGatewayRequest`) → chat/agent handlers → `commands/agent.ts` (`agentCommand`) → `agents/pi-embedded-runner/run.js` (`runEmbeddedPiAgent`). Responses are broadcast via `gateway/server-chat.ts` (`createAgentEventHandler`).

## Apps (Optional)

### macOS (Jaisiu.app)

- Menu bar control for the Gateway
- Voice Wake + push-to-talk overlay
- WebChat + debug tools
- Remote gateway control

### iOS Node

- Pairs as a node via the Bridge
- Voice trigger forwarding + Canvas surface
- Controlled via `jaisiu nodes …`

### Android Node

- Pairs via the same Bridge + pairing flow
- Exposes Canvas, Camera, and Screen capture commands

## Configuration

Config file: `~/.openclaw/openclaw.json`. Minimal example:

```json5
{
  agent: {
    model: "anthropic/claude-opus-4-5",
  },
  employees: {
    enabled: true,
  },
}
```

## Security

- **Default:** Tools run on the host for the **main** session
- **Sandboxing:** Set `agents.defaults.sandbox.mode: "non-main"` for Docker sandboxes; for Fleet MVP per-tab isolation use [docs/config/fleet-mvp-sandbox-preset.json5](docs/config/fleet-mvp-sandbox-preset.json5) and [docs/install/fleet-mvp-session-environments.md](docs/install/fleet-mvp-session-environments.md)
- **Channel Access:** Employee agents respect channel permissions from their configuration
- **DM Pairing:** Unknown senders receive pairing codes by default

## Development

```bash
git clone https://test.pryzmat.pl/root/jaisiu.git
cd jaisiu

pnpm install
pnpm ui:build
pnpm build

pnpm jaisiu onboard --install-daemon
# or: node scripts/run-node.mjs onboard --install-daemon

# Dev loop (auto-reload)
pnpm gateway:watch
```

## HR Sync Connectors

Jaisiu supports pluggable HR connectors:

- **Mock** — For testing and development
- **File** — JSON/CSV file-based sync
- **Custom** — Implement `HrSyncConnector` interface for your HR system

### Creating a Custom Connector

Implement `HrSyncConnectorBase` (see `src/employees/sync/`). Example shape:

```typescript
import { HrSyncConnectorBase } from "jaisiu";

export class MyHrConnector extends HrSyncConnectorBase {
  id = "my-hr-system";
  name = "My HR System";
  version = "1.0.0";
  supportsBidirectional = true;

  async testConnection(config) {
    // Test connection to your HR system
  }

  async syncEmployees(config) {
    // Fetch and return employees from your HR system
  }

  async getEmployee(id, config) {
    // Get single employee by ID
  }

  getConfigSchema() {
    // Return Zod schema for connector configuration
  }
}
```

## API

### Gateway RPC Methods (Employees)

- `employees.list` — List all employee agents
- `employees.get` — Get employee by ID
- `employees.update` — Update employee data
- `employees.delete` — Remove employee from agent
- `employees.channels.list` — List employee channel access
- `employees.channels.add` — Add channel access
- `employees.channels.remove` — Remove channel access
- `employees.sync.connectors.list` — List available HR connectors
- `employees.sync.connectors.test` — Test connector connection
- `employees.sync.run` — Run manual HR sync
- `employees.config.update` — Update employees configuration

## Shared YouTrack MCP (Prism-wide)

Jaisiu now includes a shared YouTrack MCP layer designed for multi-Prism use. It provides:

- shared tools (`youtrack_search`, `youtrack_get`, `youtrack_create`, `youtrack_update`, `youtrack_transition`, `youtrack_comment`, `youtrack_link`)
- Prism-aware project policy (read/write allowlists)
- idempotent writes by `requestId`
- webhook fanout into shared memory (`project:<KEY>:youtrack:events:*`)

### Bare metal setup

Set these environment variables before starting Jaisiu/Jaisiu:

```bash
export PRISM_ID=pryzm_at_bot
export PRISM_REDIS_HOST=127.0.0.1
export PRISM_REDIS_PORT=6379
export PRISM_REDIS_PASSWORD=""
export PRISM_REDIS_DB=0
export PRISM_REDIS_KEY_PREFIX=pryzm:prism:

export YOUTRACK_BASE_URL="https://youtrack.example.com"
export YOUTRACK_TOKEN="perm:..."
export YOUTRACK_WEBHOOK_SECRET="replace-me"
export YOUTRACK_PROJECT_ACCESS_JSON='{"pryzm_at_bot":{"read":["OPS","CARG"],"write":["OPS"]}}'
```

Then initialize from code:

```typescript
import { setupYouTrackSharedMcp } from "jaisiu";

const youtrack = await setupYouTrackSharedMcp({ prismId: process.env.PRISM_ID! });
```

### Docker setup

Step-by-step: [docs/deploy/COMPLETE-DEPLOY-GUIDE.md](docs/deploy/COMPLETE-DEPLOY-GUIDE.md). Verify: `./scripts/verify-jaisiu-stack.sh`.

In Docker Compose, set the same env vars (`YOUTRACK_BASE_URL`, `YOUTRACK_TOKEN`, `YOUTRACK_PROJECT_ACCESS_JSON`, `YOUTRACK_WEBHOOK_SECRET`, and Prism Redis vars). The included `docker-compose.yml` forwards them into `openclaw-gateway` and `openclaw-cli`. Local Redis/EFSS overrides: `docker-compose.override.yml`.

### Self-Improvement Tests

The self-improvement tests verify the full YouTrack → agent → code → push loop before a job can ship.

The fleet self-improvement system (YouTrack → agent → code → push) has test coverage in `src/agents/fleet-self-improvement/`. Run the suite with:

```bash
pnpm vitest run src/agents/fleet-self-improvement/ --reporter verbose
```

Key test files: `job-sync.test.ts` (job state machine), `youtrack-task.test.ts` (YouTrack wiring), `start.test.ts` (worker launch), `verify-self-improvement-outcome.test.ts` (git diff verification).

---

## Quick Start

Runtime: **Node ≥ 22**. The CLI and package name are **jaisiu** (this repo is the Jaisiu platform).

```bash
# Canonical install — EFSS TUI installer (handles Node, npm, onboarding)
curl -fsSL https://pryzm.at/efss/jaisiu/install.sh | bash

# Already have Node 22+? Skip the installer and pin a release:
npm install -g jaisiu@latest

# Bring up the gateway (install + register as OS service in one shot)
jaisiu install --install-daemon
jaisiu gateway --port 18789

# Talk to the assistant
open http://127.0.0.1:18789/                 # WebChat / Control UI
jaisiu agent --message "Ship checklist"       # headless
```

Building from source (contributors only):

```bash
pnpm install && pnpm ui:build && pnpm build
```

Config file defaults:

- New installs: `~/.jaisiu/state/jaisiu.json`
- Legacy installs: `~/.openclaw/openclaw.json`
- Override: `JAISIU_CONFIG_PATH=/path/to/file.json5`

See [`docs/reference/configuration-paths.md`](docs/reference/configuration-paths.md) for the full state-dir matrix and migration notes.

For everything else — channels, pairing, EFSS, security, deploy, architecture — **go to [docs/index.md](docs/index.md).**

---

## What Jaisiu ships

| Subsystem                         | Where                                 | What                                                                                               |
| --------------------------------- | ------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Gateway (WebSocket control plane) | `src/gateway/`                        | Single WS hub for clients, tools, events                                                           |
| Channels                          | `src/channels/`                       | WhatsApp, Telegram, Discord, Slack, Signal, iMessage, Matrix, Teams, Google Chat, Mattermost, more |
| Employee Management               | `src/employees/`                      | AI agents as employees with HR-sync connectors                                                     |
| Nodes (iOS / Android)             | `src/node-host/`                      | Paired devices, Canvas, camera, screen record, location                                            |
| EFSS portal                       | `efss-portal/` + `packages/pryzm.at`  | Enterprise file portal, branded viewer, optional TRELLIS                                           |
| macOS app                         | `apps/macos/`                         | Menu bar control, Voice Wake, push-to-talk                                                         |
| Self-improvement loop             | `src/agents/fleet-self-improvement/`  | YouTrack → code → push, JAISIU-Bench, self-upgrade harness                                         |
| Prism / MemPrism                  | `src/memprism/`, `src/shared-memory/` | Redis-backed cross-agent state and recall                                                          |
| Floci                             | `src/floci/`                          | Crowd orchestrator for multiple agent harnesses                                                    |

The **architecture SoT** is [`docs/architecture-summary.md`](docs/architecture-summary.md); a deeper code-tree snapshot lives at [`docs/CURRENT_ARCHITECTURE.md`](docs/CURRENT_ARCHITECTURE.md). When the two disagree, `architecture-summary.md` wins.

---

## Channels

| Channel         | Doc                                                        | Source       |
| --------------- | ---------------------------------------------------------- | ------------ |
| WhatsApp        | [docs/channels/whatsapp.md](docs/channels/whatsapp.md)     | Baileys      |
| Telegram        | [docs/channels/telegram.md](docs/channels/telegram.md)     | grammY       |
| Discord         | [docs/channels/discord.md](docs/channels/discord.md)       | discord.js   |
| Slack           | [docs/channels/slack.md](docs/channels/slack.md)           | Bolt         |
| Signal          | [docs/channels/signal.md](docs/channels/signal.md)         | signal-cli   |
| iMessage        | [docs/channels/imessage.md](docs/channels/imessage.md)     | imsg (macOS) |
| Microsoft Teams | [docs/channels/msteams.md](docs/channels/msteams.md)       | extension    |
| Matrix          | [docs/channels/matrix.md](docs/channels/matrix.md)         | extension    |
| Google Chat     | [docs/channels/googlechat.md](docs/channels/googlechat.md) | Chat API     |
| Mattermost      | [docs/channels/mattermost.md](docs/channels/mattermost.md) | extension    |

Full list (Line, Feishu, Zalo, Nostr, Twitch, Nextcloud Talk, etc.) at [`docs/channels/`](docs/channels/).

---

## Documentation map

| Audience                        | Start here                                                                                                                                                          |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Newcomers**                   | [`docs/index.md`](docs/index.md) — front door                                                                                                                       |
| **Install / deploy**            | [`docs/start/getting-started.md`](docs/start/getting-started.md) → [`docs/deploy/COMPLETE-DEPLOY-GUIDE.md`](docs/deploy/COMPLETE-DEPLOY-GUIDE.md)                   |
| **Configuration**               | [`docs/reference/configuration-paths.md`](docs/reference/configuration-paths.md) (canonical SoT) + [`docs/gateway/configuration.md`](docs/gateway/configuration.md) |
| **Architecture / contributors** | [`docs/book/README.md`](docs/book/README.md) (the Jaisiu Book) → [`docs/architecture-summary.md`](docs/architecture-summary.md)                                     |
| **Help / support**              | [`docs/help/index.md`](docs/help/index.md) — symptom-first flow                                                                                                     |
| **CLI reference**               | [`docs/cli/`](docs/cli/) — every subcommand                                                                                                                         |
| **Concepts**                    | [`docs/concepts/`](docs/concepts/) — sessions, memory, agent loop, OAuth, model failover, …                                                                         |

When something here disagrees with code: **code wins** until a doc PR lands.

---

## Self-improvement & the harness loop

The `fleet-self-improvement` system (YouTrack → agent → code → push) is implemented under `src/agents/fleet-self-improvement/`. The self-upgrade loop extends it with a JAISIU-Bench scorer — see [`docs/architecture/sota-swarm-upgrade.md`](docs/architecture/sota-swarm-upgrade.md) and ticket [JAISIU-196](https://pryzmat.youtrack.cloud/issues/JAISIU-196). Phase 2 (Quantum Palace as reasoning substrate) is documented but **not yet shipped** — see [`docs/tickets/JAISIU-BENCH.md`](docs/tickets/JAISIU-BENCH.md).

---

## License

MIT — see [LICENSE](LICENSE).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). AI/vibe-coded PRs welcome 🤖
