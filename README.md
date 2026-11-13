# 🧠 skills

A personal collection of **Agent Skills** — reusable, versioned instruction sets for AI coding assistants (Claude, Cursor, GitHub Copilot, and more).

Installable with the [`skills` CLI](https://github.com/vercel-labs/skills). [skills.sh](https://skills.sh) listings depend on CLI installation telemetry.

---

## What are Skills?

Skills are markdown-based instruction files (`SKILL.md`) that teach AI agents _how_ to do something — architectural patterns, coding standards, integration workflows, and more.

When you install a skill, your AI agent gets precise, opinionated context exactly when it needs it, without bloating the entire project context.

---

## 📦 Available Skills

Skills include discovery metadata. **Draft** skills contain starter guidance and are being developed one by one with the repository owner; they are installable but their project-specific standards are unfinished. Unmarked skills contain existing guidance, not a certification of quality.

### ⚛️ Frontend

| Skill | Description |
|---|---|
| `react-quality-rules` | Component design, hooks, state, rendering, and naming conventions for React |
| `react-hook-form-with-zod` | Controller pattern, Zod schemas, file-per-form separation, type-safe forms |
| `tanstack-api-integration` | TanStack Query patterns for API integration |
| `tanstack-table` (draft) | TanStack Table setup, column definitions, sorting and filtering |
| `typescript-instincts` | Advanced TypeScript patterns and strict type safety rules |
| `taste-design` | Visual hierarchy, typography, and Vayu UI screen design |
| `design-system` | Design tokens, component variants, and theming conventions |
| `animation` (draft) | CSS and JS animation patterns |
| `gsap` (draft) | GSAP animation techniques and best practices |
| `react-a11y` (draft) | Accessibility rules and ARIA patterns for React |
| `react-i18n` (draft) | Internationalization setup and conventions |
| `react-documentor` (draft) | Component documentation standards |
| `react-folder-structure` | Feature-based folder structure conventions for React |
| `zustand-usage` | Zustand store patterns and global state management |

### 🖥️ Backend

| Skill | Description |
|---|---|
| `api-standard-practices` | REST API design, error handling, and response conventions |
| `fastify-folder-structure` | Feature-first Fastify architecture, shared types/utilities, plugin scopes, and lifecycle |
| `fastapi-folder-structure` | Feature-first FastAPI architecture, dependencies, resource lifecycles, and testing |
| `mongodb-db-design` (draft) | MongoDB database design, schema modeling, and collection structure |
| `mongo-standard-practices` (draft) | MongoDB query patterns, indexing, and operational conventions |
| `pgsql-db-design` (draft) | PostgreSQL database design, table schema, and normalization patterns |
| `postgres-standard-practices` (draft) | PostgreSQL queries, migrations, and operational conventions |
| `rabbitmq-standard-practices` (draft) | RabbitMQ exchanges, queues, routing keys, and message patterns |
| `redis-standard-practices` (draft) | Redis caching strategies, key naming, and TTL patterns |

### 🔄 Full Stack

| Skill | Description |
|---|---|
| `ws-standard-practices` | WebSocket/Socket.IO structure for backend and React/Next.js: lifecycle, event services, hooks, and shared contracts |

The unified WebSocket skill replaces `ws-be-standard-practices` and `ws-fe-standard-practices`. Existing consumers should install the unified skill and remove their old installed copies; updating this repository does not remove consumer-local installations.

### 🤖 AI / Agent

| Skill | Description |
|---|---|
| `token-efficiency-mode` (draft) | Patterns to reduce token usage while maintaining output quality |

---

## 🚀 Quick Start — Install a Skill

You **don't need to clone this repo**. Use the `skills` CLI via `npx` to install any skill directly into your project.

### Install from this repo (interactive)

```bash
npx skills add Rugved1652/skills
```

This launches an interactive prompt where you:
1. Choose which skill(s) to install
2. Select your AI agent (Claude, Cursor, Copilot, etc.)
3. Choose global or local installation

---

## 🛠 CLI Reference

```bash
# Browse and install skills from this repo
npx skills add Rugved1652/skills

# List all installed skills in your project
npx skills list

# Search for skills by keyword
npx skills find react
npx skills find typescript

# Update all installed skills
npx skills update

# Remove a skill
npx skills remove <skill-name>
```

---

## 📂 Repo Structure

```
skills/
├── skills/
│   ├── gsap/
│   │   └── SKILL.md
│   ├── animation/
│   │   └── SKILL.md
│   ├── api-standard-practices/
│   │   └── SKILL.md
│   ├── design-system/
│   │   └── SKILL.md
│   ├── fastapi-folder-structure/
│   │   └── SKILL.md
│   ├── fastify-folder-structure/
│   │   └── SKILL.md
│   ├── mongodb-db-design/
│   │   └── SKILL.md
│   ├── mongo-standard-practices/
│   │   └── SKILL.md
│   ├── pgsql-db-design/
│   │   └── SKILL.md
│   ├── postgres-standard-practices/
│   │   └── SKILL.md
│   ├── react-a11y/
│   │   └── SKILL.md
│   ├── react-documentor/
│   │   └── SKILL.md
│   ├── react-folder-structure/
│   │   └── SKILL.md
│   ├── react-hook-form-with-zod/
│   │   └── SKILL.md
│   ├── react-i18n/
│   │   └── SKILL.md
│   ├── react-quality-rules/
│   │   └── SKILL.md
│   ├── rabbitmq-standard-practices/
│   │   └── SKILL.md
│   ├── redis-standard-practices/
│   │   └── SKILL.md
│   ├── tanstack-api-integration/
│   │   └── SKILL.md
│   ├── taste-design/
│   │   └── SKILL.md
│   ├── tanstack-table/
│   │   └── SKILL.md
│   ├── token-efficiency-mode/
│   │   └── SKILL.md
│   ├── typescript-instincts/
│   │   └── SKILL.md
│   ├── ws-standard-practices/
│   │   ├── SKILL.md
│   │   └── references/
│   │       └── folder-structures.md
│   └── zustand-usage/
│       └── SKILL.md
├── LICENSE
└── README.md
```

Every skill lives in its **own named directory** with a `SKILL.md` entry point. This keeps each skill self-contained and makes it easy to add supporting files (examples, snippets, diagrams) alongside `SKILL.md` in the future.

---

## ⚙️ How Skills Work

```
GitHub: Rugved1652/skills
      │
      ▼
npx skills add Rugved1652/skills
      │
      ├── Reads each skill's SKILL.md
      ├── Copies to agent config directory
      │     ├── Claude  →  .claude/skills/
      │     ├── Cursor  →  .cursor/skills/
      │     └── Copilot →  .github/skills/
      │
      └── Agent loads skill only when relevant (no context bloat)
```

Skills use a **lazy-load** model — your agent reads the skill instructions only when the task matches, keeping prompts lean and focused.

---

## Discovery and listing

Preview the skills available from GitHub without installing them:

```bash
npx skills add Rugved1652/skills --list
```

To install all skills, including drafts:

```bash
npx skills add Rugved1652/skills --skill '*'
```

The [skills.sh FAQ](https://skills.sh/docs/faq) explains that listings are created automatically from CLI installation telemetry. Uploading files to GitHub or running `--list` alone does not register an installation. Telemetry-disabled installations do not contribute to listing metrics. The FAQ does not promise a listing update time.

---

## ✅ Verify Installation

After installing, ask your AI agent:

> _"What skills do you have access to right now?"_

The agent will list all loaded skills and confirm they are active.

---

## 🔗 Links

- [skills.sh Registry](https://skills.sh) — Browse all published skills
- [Vercel Agent Skills Docs](https://vercel.com/docs/agent-skills) — Official documentation
- [skills CLI on npm](https://www.npmjs.com/package/skills) — CLI reference

---

## 📄 License

MIT — see [LICENSE](./LICENSE)
