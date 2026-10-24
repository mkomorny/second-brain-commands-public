---
name: second-brain-commands
description: Standardized slash commands and operational triggers for the Second Brain MCP across all AI apps (Claude, Grok, Antigravity/Gemini, Cursor). Provides seamless semantic recall, note capture, updates, and graph traversal while strictly enforcing the mandatory pre-edit approval policy.
---

# Skill: Universal Second Brain Commands (`second-brain-commands`)

This skill defines a universal command syntax and operational protocol for interacting with the user's personal Second Brain MCP server (`second-brain`) across any AI assistant (Claude Desktop, Claude Code, Grok CLI, Antigravity/Gemini, Cursor, etc.).

---

## 1. Quick Command Reference

| Command | Shorthand | Type | MCP Tool | Approval Required? |
| :--- | :--- | :--- | :--- | :--- |
| `/brain recall <query>` | `/recall <query>` | Read | `recall` | ❌ No (executes immediately) |
| `/brain get <id>` | `/get <id>` | Read | `get` | ❌ No (executes immediately) |
| `/brain recent [limit]` | `/recent [limit]` | Read | `list_recent` | ❌ No (executes immediately) |
| `/brain connections <id>` | `/connections <id>` | Read | `connections` | ❌ No (executes immediately) |
| `/brain save <text>` | `/remember <text>` | Write | `remember` | ⚠️ **YES (Mandatory pre-approval)** |
| `/brain append <id> <text>` | `/append <id> <text>` | Write | `append` | ⚠️ **YES (Mandatory pre-approval)** |
| `/brain update <id> <text>` | `/update <id> <text>` | Write | `update` | ⚠️ **YES (Mandatory pre-approval)** |
| `/brain status <id> <status>` | `/status <id> <status>` | Write | `set_status` | ⚠️ **YES (Mandatory pre-approval)** |
| `/brain link <id1> <id2> [type]` | `/link <id1> <id2>` | Write | `link` | ⚠️ **YES (Mandatory pre-approval)** |
| `/brain unlink <id1> <id2>` | `/unlink <id1> <id2>` | Write | `unlink` | ⚠️ **YES (Mandatory pre-approval)** |
| `/brain forget <id>` | `/forget <id>` | Write | `forget` | ⚠️ **YES (Mandatory pre-approval)** |
| `/brain sync [topic]` | `/brain context` | Macro | `recall` + `list_recent` | ❌ No (read-only briefing) |
| `/brain digest` | — | Macro | `list_recent` + `recall` | ❌ No (read-only briefing) |
| `/brain help` | — | Help | — | ❌ No (prints command guide) |

---

## 2. Command Specifications & Behavior

### Retrieval Commands (Read-Only)

#### `/brain recall <query>` or `/recall <query>`
- **Intent**: Semantically searches stored memories for context, past decisions, facts, or preferences.
- **Parameters & Flags**:
  - `<query>`: Natural language topic. The AI automatically expands this into an intent-framed search query (e.g. `User wants to [action] regarding [topic] — what should I know?`).
  - `--hops <0-2>` or `-h <0-2>`: Knowledge graph expansion depth (default: 0; use 1–2 when tracing causes, outcomes, history, or evolution).
  - `--tag <tag>` or `-t <tag>`: Filter results by an exact tag (e.g. `task`, `work`, `ableton`).
  - `--kind <semantic|episodic>` or `-k <kind>`: Filter to durable facts/preferences (`semantic`) or past events/sessions (`episodic`).
  - `--topK <1-20>`: Number of results to return (default: 5).
- **Execution**: The AI invokes `recall` immediately. If any returned result ends in `[truncated ...]`, the AI alerts the user and recommends `/brain get <id>` to view the full text.

#### `/brain get <id>` or `/get <id>`
- **Intent**: Retrieves the complete, untruncated content and metadata of a specific memory by its unique ID.
- **Execution**: Invokes `get(id: "<id>")` immediately.

#### `/brain recent [limit]` or `/recent [limit]`
- **Intent**: Lists the most recently recorded memories in reverse-chronological order.
- **Parameters**: `limit` (optional integer, defaults to 5, max 20).
- **Execution**: Invokes `list_recent(limit: <limit>)` immediately.

#### `/brain connections <id>` or `/connections <id>`
- **Intent**: Surfaces all memories linked directly to the specified entry in the knowledge graph.
- **Execution**: Invokes `connections(id: "<id>")` immediately.

---

### Mutation Commands (STRICT Pre-Approval Required)

> [!IMPORTANT]
> **MANDATORY APPROVAL POLICY**:
> You must **NEVER** call `remember`, `append`, `update`, `set_status`, `link`, `unlink`, or `forget` silently or automatically.
> When the user runs any mutation command, the AI **MUST** halt and display the exact approval proposal block below. The AI must wait for the user to confirm ("yes", "confirm", "proceed", etc.) before invoking the MCP tool.

#### Approval Proposal Template
Whenever a mutation is triggered, output this exact structure:

```text
📋 Proposed Second Brain Action:
- Tool: [remember | append | update | set_status | link | unlink | forget]
- Target ID: [Entry ID or "NEW"]
- Exact Content / Addition: "[Exact text to be sent]"
- Tags: [e.g. "work", "ableton", "task"]
- Volatility: [durable | state | volatile]
- Source: [claude | grok | antigravity]
- Rationale: [Clear, concise reason why this should be preserved/modified]

👉 Do you approve this second brain update?
```

#### `/brain save <text>` or `/remember <text>`
- **Intent**: Stores a new distinct, durable idea, decision, preference, or fact.
- **Inline Metadata Parsing**:
  - Tags: `#<tag>` or `--tags <tag1,tag2>` (e.g. `/brain save Chose WebView2 for UI #live-actions #decision`).
  - Volatility: `!durable`, `!state`, or `!volatile` (e.g. `!durable` for permanent architectural choices).
- **Execution**: Parses inline tags and volatility, infers the current client platform for `source`, formats the proposal block, and awaits user confirmation before calling `remember`.

#### `/brain append <id> <text>` or `/append <id> <text>`
- **Intent**: Appends new progress, updates, or outcomes to an existing memory without overwriting original content.
- **Execution**: Checks existing entry if needed, presents proposal block with the exact addition and target ID, and awaits user approval before calling `append`.

#### `/brain update <id> <text>` or `/update <id> <text>`
- **Intent**: Fully replaces the content of an existing memory when information is obsolete or reversed.
- **Execution**: Fetches existing memory via `get(id)` to show the user before/after diff, presents proposal block, and awaits user approval before calling `update`.

#### `/brain status <id> <canonical|draft|deprecated>`
- **Intent**: Updates lifecycle status:
  - `canonical`: Authoritative and protected from overwrite.
  - `draft`: Tentative or work-in-progress.
  - `deprecated`: Outdated; excluded from semantic recall but preserved for audit.
- **Execution**: Displays proposal block explaining the consequence and awaits user approval before calling `set_status`.

#### `/brain link <source_id> <target_id> [rel_type]`
- **Intent**: Creates an explicit relationship edge between two memories in the graph.
- **Allowed Types**: `relates_to` (default), `supersedes`, `caused_by`, `decided`, `about_person`, `part_of_project`, `follows`, `drawn_from`.
- **Execution**: Displays proposal block showing both IDs and relationship type, awaiting user approval before calling `link`.

#### `/brain unlink <source_id> <target_id>`
- **Intent**: Removes an existing connection between two memories.
- **Execution**: Displays proposal block and awaits approval before calling `unlink`.

#### `/brain forget <id>` or `/forget <id>`
- **Intent**: Permanently deletes an entry by ID.
- **Execution**: Fetches entry first via `get(id)`, displays the exact memory to be deleted, warns that deletion is permanent, and awaits explicit user confirmation before calling `forget`.

---

### Workflow & Macro Commands

#### `/brain sync [topic]` or `/brain context [topic]`
- **Intent**: Conversation-opening context briefing.
- **Execution**:
  1. Identifies current project or topic from argument (or active conversation).
  2. Calls `recall` with intent: `User is working on [topic] — retrieve active tasks, key decisions, and user preferences`.
  3. Formats results into a clean, 1-screen briefing covering:
     - 📌 **Key Decisions & Rules**
     - ⚙️ **Active Preferences & Setup**
     - 📝 **Open Tasks & Follow-ups**

#### `/brain digest`
- **Intent**: Cross-project or account-wide executive summary.
- **Execution**:
  1. Calls `list_recent(limit: 15)`.
  2. Synthesizes recent items into categories: Recent Decisions, Open Tasks, and New Context.

#### `/brain help`
- **Intent**: Prints the quick command table and parameter examples directly into chat.

---

## 3. Universal Client Compatibility

When operating in different AI clients, set the `source` field accordingly:
- Claude Desktop / Claude Code: `source: "claude"`
- Grok CLI: `source: "grok"`
- Antigravity / Gemini: `source: "antigravity"`
- Generic / Other: `source: "desktop"`

---

## 4. Invariant Safety Rules
1. **Never Call Mutation Tools Silently**: Always present the proposal and obtain user approval.
2. **Connector Protection**: Never modify any files in `workspace/second-brain-cloudflare` or change MCP server configurations without explicit permission.
3. **Intent Framing**: Never send bare keyword searches to `recall`; always frame with subject and intent.
