# ABAP Agentic Setup for Claude Code

A two-level hierarchical agent setup for Claude Code, tailored for **ABAP Cloud / SAP Steampunk** and **SAP Fiori** development. The main orchestrator delegates specialized tasks to purpose-built subagents, each running on the most cost-effective model for its role.

## Overview

Rather than running a single Claude Code session for everything, this setup splits work across focused agents:

- The **main agent** (Sonnet 4.6) plans, coordinates, and synthesizes
- **Subagents** handle exploration, architecture, writing, and validation — each with restricted tools and the right model for the job

This keeps the main context window clean, reduces token costs on repetitive tasks, and enforces a disciplined development workflow.

## Architecture

```
Main Orchestrator (Sonnet 4.6)
├── codebase-explorer   (Haiku)   — read-only SAP & local filesystem exploration
├── abap-syntax-checker (Haiku)   — syntax validation & ATC checks
├── rap-odata-analyzer  (Sonnet)  — RAP/CDS architecture analysis & design
└── abap-code-writer    (Sonnet)  — writes & pushes ABAP code to SAP system
```

### Standard workflow

```
1. EXPLORE   → codebase-explorer     understand what exists before touching anything
2. ANALYZE   → rap-odata-analyzer    for RAP/OData tasks: design before coding
3. PLAN      → main agent            decide what to build; confirm with user if needed
4. WRITE     → abap-code-writer      implement and push to SAP
5. VALIDATE  → abap-syntax-checker   syntax check + ATC before activation
6. REVIEW    → main agent            synthesize and report to user
```

## Subagents

### `codebase-explorer` — Haiku

Dual-mode read-only explorer. Covers both the live SAP system (via MCP) and local Fiori/UI5 project files (via filesystem tools).

**SAP capabilities:**
- List objects in packages and sub-packages
- Read source code of any ABAP object (class, program, CDS view, BDEF, SRVD)
- Find objects by name pattern and type
- Trace where-used / find references
- Explore CDS view dependency hierarchies
- Inspect transport assignments

**Local filesystem capabilities:**
- Map Fiori app project structure
- Find UI5 views, controllers, fragments, i18n files
- Connect a Fiori app back to its RAP backend via `manifest.json`

### `abap-syntax-checker` — Haiku

Validates code before it reaches the SAP system. Runs against the live system via MCP.

- `SyntaxCheck` for immediate syntax errors
- `RunATCCheck` for full ATC quality analysis (Priority 1=Error, 2=Warning, 3=Info)
- `PrettyPrint` to normalize formatting before validation
- Manual cross-check for ABAP Cloud restricted syntax violations

### `rap-odata-analyzer` — Sonnet

Architecture specialist for the full RAP stack.

**Analysis mode:** reads and explains existing CDS hierarchies, behavior definitions, service definitions, entity relationships, and flags anti-patterns.

**Design mode:** produces a layered scaffolding plan for new OData V4 services — from base interface view through projection, metadata extension, behavior definition, and service binding.

### `abap-code-writer` — Sonnet

Implements and pushes code to the SAP system.

- Writes ABAP classes, interfaces, programs, CDS views, BDEFs, SRVDs, metadata extensions
- Edits existing objects surgically via `EditSource` (preferred for small changes)
- Handles local Fiori/UI5 files (JS, XML, JSON) via filesystem tools
- Selects the right MCP write tool automatically based on the object type

## Prerequisites

- [Claude Code](https://claude.ai/code) installed and authenticated (Pro, Max, Teams, or Enterprise plan)
- [vibing-steampunk (vsp)](https://github.com/oisee/vibing-steampunk) MCP server configured as `vsp`
- [fr0ster/mcp-abap-adt](https://github.com/fr0ster/mcp-abap-adt) MCP server configured as `abap-adt` (fallback)
- Both MCP servers connected to your SAP ABAP system

### MCP server names

The agent `tools:` fields use `mcp__vsp__*` and `mcp__abap-adt__*` prefixes. These must match the server names registered in your Claude Code configuration. Verify with:

```bash
claude mcp list
```

If your servers are registered under different names, update the `tools:` field in each agent file accordingly.

## Installation

1. Clone this repository into your project folder:

```bash
git clone https://github.com/<your-username>/abap-agentic-setup.git
cd abap-agentic-setup
```

Or copy the files manually into an existing project:

```
your-project/
├── CLAUDE.md                          ← orchestrator instructions (project root)
└── .claude/
    └── agents/
        ├── codebase-explorer.md
        ├── abap-syntax-checker.md
        ├── rap-odata-analyzer.md
        └── abap-code-writer.md
```

2. Open Claude Code in your project directory:

```bash
cd your-project
claude
```

3. Claude Code will automatically detect `CLAUDE.md` and the agents in `.claude/agents/`.

## Usage examples

Once set up, you can drive the full workflow with natural language:

```
Explore package ZNORTH_EXT and give me an overview of what's there.
```

```
Design a new RAP service for purchase order approvals — managed, draft-enabled, OData V4.
```

```
Implement the CDS interface view ZI_PURCHASE_ORDER based on table EKKO.
```

```
Check the syntax of ZCL_MY_CLASS and run ATC before I activate it.
```

Claude Code will automatically delegate each step to the appropriate subagent.

## Customization

### Adapting to your project

Edit `CLAUDE.md` to add project-specific context: package names, naming conventions, transport numbers, team conventions, or links to architecture documentation.

### Adjusting models

Each agent's `model:` field in the frontmatter can be changed independently:
- `haiku` — fastest and cheapest; suitable for search and validation tasks
- `sonnet` — balanced; good for architecture and code generation
- `opus` — maximum capability; use for the most complex reasoning tasks

### Adding more agents

Create additional `.md` files in `.claude/agents/` following the same frontmatter format. See the [Claude Code subagents documentation](https://code.claude.com/docs/en/sub-agents) for full reference.

## Related tools

- [vibing-steampunk](https://github.com/oisee/vibing-steampunk) — ADT-to-MCP bridge used as primary SAP connection
- [Claude Code documentation](https://code.claude.com/docs/en/overview)
- [SAP RAP documentation](https://help.sap.com/docs/abap-cloud/abap-rap/abap-restful-application-programming-model)
