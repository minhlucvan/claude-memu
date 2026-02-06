<h1 align="center">
  <br>
  Claude-Memu
  <br>
</h1>

<h4 align="center">Persistent memory for <a href="https://claude.com/claude-code" target="_blank">Claude Code</a> powered by <a href="https://github.com/NevaMind-AI/memU" target="_blank">memU</a>.</h4>

<p align="center">
  <a href="LICENSE">
    <img src="https://img.shields.io/badge/License-AGPL%203.0-blue.svg" alt="License">
  </a>
  <a href="package.json">
    <img src="https://img.shields.io/badge/version-1.0.0-green.svg" alt="Version">
  </a>
  <a href="package.json">
    <img src="https://img.shields.io/badge/node-%3E%3D18.0.0-brightgreen.svg" alt="Node">
  </a>
</p>

<p align="center">
  <a href="#quick-start">Quick Start</a> •
  <a href="#how-it-works">How It Works</a> •
  <a href="#configuration">Configuration</a> •
  <a href="#api">API</a> •
  <a href="#license">License</a>
</p>

---

Claude-Memu seamlessly preserves context across Claude Code sessions using [memU](https://github.com/NevaMind-AI/memU) for hierarchical memory storage. It automatically captures observations, generates summaries, and injects relevant context into future sessions.

## Quick Start

### Option A: Local Mode (No API Key Required)

Run fully locally with file-based storage:

```bash
# In Claude Code
/plugin marketplace add minhlucvan/claude-memu
/plugin install claude-memu
```

That's it! Without an API key, claude-memu automatically uses local file storage in `~/.claude-memu/data/`.

### Option B: Cloud Mode (memU API)

For advanced features like semantic search and proactive context:

#### 1. Get a memU API Key

Sign up at [api.memu.so](https://api.memu.so) or deploy a self-hosted instance.

#### 2. Install the Plugin

```bash
# In Claude Code
/plugin marketplace add minhlucvan/claude-memu
/plugin install claude-memu
```

#### 3. Configure

Set your API key in `~/.claude-memu/settings.json`:

```json
{
  "CLAUDE_MEMU_API_KEY": "your-api-key-here"
}
```

Or via environment variable:

```bash
export CLAUDE_MEMU_API_KEY="your-api-key-here"
```

#### 4. Restart Claude Code

Context from previous sessions will automatically appear in new sessions.

---

## Features

- **Dual Storage Modes** - Run locally (no API) or with memU cloud
- **Persistent Memory** - Context survives across sessions
- **Proactive Context** - LLM-powered context retrieval (API mode)
- **Semantic Search** - RAG-based memory queries (API mode)
- **Project Isolation** - Memories organized by project
- **Privacy Control** - Use `<private>` tags to exclude sensitive content
- **Fully Offline** - Local mode works without internet

---

## How It Works

```
┌─────────────────────────────────────────────────────────────┐
│                    Claude Code Plugin                        │
├─────────────────────────────────────────────────────────────┤
│  Hooks (5 Lifecycle Events)                                 │
│  Setup → SessionStart → UserPromptSubmit → PostToolUse → Stop│
├─────────────────────────────────────────────────────────────┤
│  Worker Service (Express on :37777)                         │
├─────────────────────────────────────────────────────────────┤
│  UnifiedStore (auto-selects based on API key)               │
├────────────────────────┬────────────────────────────────────┤
│  LocalStore            │  MemuStore                         │
│  (File-based JSON)     │  (memU API client)                 │
├────────────────────────┴────────────────────────────────────┤
│  ~/.claude-memu/data/  │  memU API (api.memu.so/self-hosted)│
└─────────────────────────────────────────────────────────────┘
```

### Core Components

1. **Lifecycle Hooks** - Capture tool usage at 5 lifecycle events via bun-runner
2. **Worker Service** - HTTP API on port 37777
3. **UnifiedStore** - Auto-selects between LocalStore and MemuStore
4. **LocalStore** - File-based JSON storage (no API key required)
5. **MemuStore** - memU API storage with semantic search
6. **MemuClient** - HTTP client for memU API

### Data Flow

1. **Setup** - Install dependencies (runs `setup.sh`)
2. **SessionStart** - Start worker, retrieve relevant context, capture user message
3. **UserPromptSubmit** - Start worker, create/init session
4. **PostToolUse** - Start worker, store observations
5. **Stop** - Start worker, generate and store session summary

---

## Configuration

Settings in `~/.claude-memu/settings.json`:

```json
{
  "CLAUDE_MEMU_MODE": "auto",
  "CLAUDE_MEMU_API_KEY": "your-api-key",
  "CLAUDE_MEMU_API_URL": "https://api.memu.so",
  "CLAUDE_MEMU_NAMESPACE": "default",
  "CLAUDE_MEMU_WORKER_PORT": "37777",
  "CLAUDE_MEMU_WORKER_HOST": "127.0.0.1",
  "CLAUDE_MEMU_DATA_DIR": "~/.claude-memu",
  "CLAUDE_MEMU_LOG_LEVEL": "INFO",
  "CLAUDE_MEMU_CONTEXT_LIMIT": "20",
  "CLAUDE_MEMU_PROACTIVE_CONTEXT": "true",
  "CLAUDE_MEMU_SHOW_SUMMARIES": "true"
}
```

### Storage Modes

| Mode | Description |
|------|-------------|
| `auto` | Use API if key provided, otherwise local (default) |
| `api` | Force memU API mode (requires API key) |
| `local` | Force local file-based storage |

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `CLAUDE_MEMU_MODE` | Storage mode (`auto`, `api`, `local`) | `auto` |
| `CLAUDE_MEMU_API_KEY` | memU API key (optional for local mode) | - |
| `CLAUDE_MEMU_API_URL` | memU API URL | `https://api.memu.so` |
| `CLAUDE_MEMU_NAMESPACE` | Namespace for isolation | `default` |
| `CLAUDE_MEMU_WORKER_PORT` | Worker service port | `37777` |
| `CLAUDE_MEMU_WORKER_HOST` | Worker service host | `127.0.0.1` |
| `CLAUDE_MEMU_DATA_DIR` | Data directory location | `~/.claude-memu` |
| `CLAUDE_MEMU_LOG_LEVEL` | Log verbosity (DEBUG, INFO, WARN, ERROR) | `INFO` |
| `CLAUDE_MEMU_CONTEXT_LIMIT` | Max items in context | `20` |
| `CLAUDE_MEMU_CONTEXT_TYPES` | Observation types to include | all types |
| `CLAUDE_MEMU_CONTEXT_CONCEPTS` | Concepts to include | all concepts |
| `CLAUDE_MEMU_PROACTIVE_CONTEXT` | Enable proactive context (API mode) | `true` |
| `CLAUDE_MEMU_SHOW_SUMMARIES` | Show summaries in context | `true` |

---

## API

### UnifiedStore (Recommended)

```typescript
import { initializeStore } from './services/memu';

// Auto-selects local or API mode based on API key
const store = await initializeStore();

console.log(store.getMode()); // 'local' or 'api'
console.log(store.isLocalMode()); // true/false

// Sessions
const session = await store.createSession(contentSessionId, project, prompt);

// Observations
await store.storeObservation(memorySessionId, project, {
  type: 'decision',
  title: 'Chose React over Vue',
  facts: ['Better TypeScript support', 'Larger ecosystem'],
  concepts: ['architecture', 'framework'],
  filesModified: ['package.json'],
});

// Summaries
await store.storeSummary(memorySessionId, project, {
  request: 'Set up frontend framework',
  completed: 'Installed React with TypeScript',
  learned: 'React has better TS integration',
});

// Search
const results = await store.search({
  text: 'authentication',
  project: 'my-project',
  method: 'rag',
  limit: 20,
});

// Context Injection
const context = await store.getContextForProject(project, 10);
```

### MemuClient

```typescript
import { createMemuClient } from './services/memu';

const client = createMemuClient({
  apiKey: 'your-key',
  apiUrl: 'https://api.memu.so',
  namespace: 'default',
});

// Retrieve memories
const response = await client.retrieve({
  queries: [{ role: 'user', content: 'authentication bugs' }],
  method: 'rag',
  limit: 20,
});

// Create memory item
await client.createItem({
  memoryType: 'decision',
  content: 'Chose JWT for auth',
  tags: ['security', 'architecture'],
});

// List categories (projects)
const categories = await client.listCategories();
```

---

## memU Concepts

| Claude-Memu | memU |
|-------------|------|
| Project | Category |
| Observation | Item |
| Summary | Item (tagged as summary) |
| User Prompt | Item (conversation type) |

### Tags Convention

- `session:{id}` - Links item to session
- `project:{name}` - Links item to project
- `type:{type}` - Observation type (decision, bugfix, etc.)
- `summary` - Marks item as session summary

---

## System Requirements

- **Node.js** 18.0.0+
- **Bun** (auto-installed)
- **memU API Key** (optional) - required only for cloud features from [api.memu.so](https://api.memu.so)

---

## Memory Browser UI

Access the web-based memory browser at **http://localhost:37777/** when the worker is running.

Features:
- **View all projects** with stored memories
- **Real-time updates** via SSE stream
- **Processing status** indicator
- **Browse observations and summaries**

The UI works with both local and API modes.

---

## Privacy

Use `<private>` tags to exclude content from storage:

```
<private>
API_KEY=secret123
</private>
```

Content within these tags is stripped at the hook layer before reaching memU.

---

## Development

```bash
# Install dependencies
npm install

# Build
npm run build

# Build and sync to marketplace
npm run build-and-sync
```

---

## License

**GNU Affero General Public License v3.0** (AGPL-3.0)

See [LICENSE](LICENSE) for details.

---

## Links

- **memU**: [github.com/NevaMind-AI/memU](https://github.com/NevaMind-AI/memU)
- **Issues**: [GitHub Issues](https://github.com/minhlucvan/claude-memu/issues)

---

**Powered by [memU](https://github.com/NevaMind-AI/memU)** | **Built for Claude Code**
