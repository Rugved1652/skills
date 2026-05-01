# 🧠 skills

A personal collection of **Agent Skills** — reusable, versioned instruction sets for AI coding assistants (Claude, Cursor, GitHub Copilot, and more).

Published on [skills.sh](https://skills.sh) · Powered by the [`skills` CLI](https://vercel.com/blog/agent-skills) by Vercel.

---

## What are Skills?

Skills are markdown-based instruction files (`SKILL.md`) that teach AI agents _how_ to do something — architectural patterns, coding standards, integration workflows, and more.

When you install a skill, your AI agent gets precise, opinionated context exactly when it needs it, without bloating the entire project context.

---

## 📦 Available Skills

### ⚛️ Frontend

| Skill | Description |
|---|---|
| `react-quality-rules` | Component design, hooks, state, rendering, and naming conventions for React |
| `react-hook-form-with-zod` | Controller pattern, Zod schemas, file-per-form separation, type-safe forms |
| `tanstack-api-integration` | TanStack Query patterns for API integration |
| `tanstack-table` | TanStack Table setup, column definitions, sorting and filtering |
| `typescript-instincts` | Advanced TypeScript patterns and strict type safety rules |
| `design-system` | Design tokens, component variants, and theming conventions |
| `animation` | CSS and JS animation patterns |
| `GSAP` | GSAP animation techniques and best practices |
| `react-a11y` | Accessibility rules and ARIA patterns for React |
| `react-i18n` | Internationalization setup and conventions |
| `react-documentor` | Component documentation standards |
| `react-folder-structure` | Feature-based folder structure conventions for React |
| `zustand-usage` | Zustand store patterns and global state management |
| `ws-fe-standard-practices` | WebSocket client patterns and real-time state handling on the frontend |

### 🖥️ Backend

| Skill | Description |
|---|---|
| `api-standard-practices` | REST API design, error handling, and response conventions |
| `fastify-folder-structure` | Modular Fastify project structure and plugin conventions |
| `mongodb-db-design` | MongoDB database design, schema modeling, and collection structure |
| `mongo-standard-practices` | MongoDB query patterns, indexing, and operational conventions |
| `pgsql-db-design` | PostgreSQL database design, table schema, and normalization patterns |
| `postgres-standard-practices` | PostgreSQL queries, migrations, and operational conventions |
| `rabbitmq-standard-practices` | RabbitMQ exchanges, queues, routing keys, and message patterns |
| `redis-standard-practices` | Redis caching strategies, key naming, and TTL patterns |
| `ws-be-standard-practices` | WebSocket server architecture and event handling on the backend |

### 🤖 AI / Agent

| Skill | Description |
|---|---|
| `token-efficiency-mode` | Patterns to reduce token usage while maintaining output quality |

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
│   ├── GSAP/
│   │   └── SKILL.md
│   ├── animation/
│   │   └── SKILL.md
│   ├── api-standard-practices/
│   │   └── SKILL.md
│   ├── design-system/
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
│   ├── tanstack-table/
│   │   └── SKILL.md
│   ├── token-efficiency-mode/
│   │   └── SKILL.md
│   ├── typescript-instincts/
│   │   └── SKILL.md
│   ├── ws-be-standard-practices/
│   │   └── SKILL.md
│   ├── ws-fe-standard-practices/
│   │   └── SKILL.md
│   └── zustand-usage/
│       └── SKILL.md
├── LICENSE
└── README.md
```

Every skill lives in its **own named directory** with a `SKILL.md` entry point. This keeps each skill self-contained and makes it easy to add supporting files (examples, snippets, diagrams) alongside `SKILL.md` in the future.

---

## ⚙️ How Skills Work

```
skills.sh registry
      │
      ▼
npx skills add Rugved1652/skills
      │
      ├── Reads each skill's SKILL.md
      ├── Copies to agent config directory
      │     ├── Claude  →  .claude/skills/
      │     ├── Cursor  →  .cursor/skills/
      │     └── Copilot →  .github/copilot-instructions.md
      │
      └── Agent loads skill only when relevant (no context bloat)
```

Skills use a **lazy-load** model — your agent reads the skill instructions only when the task matches, keeping prompts lean and focused.

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
