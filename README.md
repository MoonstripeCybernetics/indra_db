# indra_db

**Persistent memory for AI agents.** A git-like database that lets agents record their reasoning, explore alternatives through branching, and maintain consistency across sessions.

[![Crates.io](https://img.shields.io/crates/v/indra_db.svg)](https://crates.io/crates/indra_db)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## The Problem

AI agents start fresh every session. They can't remember why they recommended something yesterday, can't maintain consistency in their reasoning, and can't explore "what if" scenarios without losing their main thread.

**indra_db** solves this with:

- 🧠 **Persistent memory** — Record reasoning that survives session boundaries
- 🌿 **Branching** — Explore alternatives without losing the main thread
- 🔍 **Semantic search** — Find past decisions by meaning, not keywords  
- 📜 **Full history** — See how understanding evolved, diff any two points

## Quick Example

```bash
# Agent records a decision
indra create "Recommended PostgreSQL for this project. User needs ACID transactions 
for e-commerce orders, and the data model is highly relational." --id db-choice

# Next session: agent searches before making recommendations
indra search "database recommendations"
# Returns the PostgreSQL decision with context

# Agent wants to explore an alternative
indra branch try-mongodb
indra checkout try-mongodb
indra create "Exploring MongoDB: Would need to handle transactions at app level..."

# Compare the two approaches
indra diff main
# Shows what's different between the branches

# Back to main reasoning
indra checkout main
```

## Branching: The Key Feature

Most memory systems are linear. Indra is git-like:

```bash
# Before exploring a risky approach
indra branch experiment
indra checkout experiment

# Explore freely - main branch is untouched
indra create "What if we used microservices instead..."
indra create "Actually, this introduces complexity X, Y, Z..."

# See what you've explored
indra diff main

# Decide it's not worth it
indra checkout main
# Original reasoning preserved, experiment kept for reference
```

**Why this matters:**
- Agents can explore alternatives without polluting their main reasoning
- You can see exactly how two approaches differ
- Failed experiments are preserved for learning, not lost
- Context-specific reasoning can live in separate branches

## Installation

```bash
# Via cargo (recommended)
cargo install indra_db

# Or download prebuilt binary from GitHub Releases
# Available for macOS, Linux, Windows (x86_64 and ARM64)
```

## CLI Reference

### Core Commands

```bash
indra init                          # Create new database
indra create "content" [--id name]  # Record an entry
indra search "query" [-l 10]        # Semantic search
indra list                          # List all entries
indra get <id>                      # Get specific entry
indra update <id> "new content"     # Update entry
```

### Branching Commands

```bash
indra branch                        # List branches
indra branch <name>                 # Create branch
indra checkout <name>               # Switch branch
indra diff [from] [to]              # Compare branches/commits
indra log                           # View commit history
```

### Sync Commands

```bash
indra login                         # Authenticate with IndraDB
indra remote add origin user/repo   # Add remote
indra push origin                   # Push to cloud
indra pull origin                   # Pull from cloud
```

## Using with AI Agents (MCP)

The primary use case is through the MCP server. Install it:

```bash
bun add -g indra_db_mcp
```

Then configure your agent (Claude Code example):

```markdown
# In CLAUDE.md
@import node_modules/indra_db_mcp/INDRA_INSTRUCTIONS.md
```

The agent gets these tools:
- `indra_remember` — Record reasoning and decisions
- `indra_search` — Find past reasoning by meaning
- `indra_branch` — Create/switch/list branches
- `indra_experiment` — Quick branch creation for exploration
- `indra_diff` — Compare branches or points in history
- `indra_history` — View evolution of reasoning

See [indra_db_mcp](https://github.com/MoonstripeCybernetics/indra_db_mcp) for full documentation.

## Library Usage

```rust
use indra_db::{Database, embedding::HFEmbedder};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    // Open with local embeddings
    let embedder = HFEmbedder::new("sentence-transformers/all-MiniLM-L6-v2").await?;
    let mut db = Database::open_or_create(".indra")?.with_embedder(embedder);

    // Record reasoning
    db.create_thought_with_id("arch-decision", 
        "Recommended monolith for this 3-person team. Faster iteration.")?;
    db.commit("Record architecture decision")?;

    // Search by meaning
    let results = db.search("architecture recommendations", 5)?;
    
    // Branch for exploration
    db.create_branch("try-microservices")?;
    db.checkout("try-microservices")?;
    db.create_thought("What if we used microservices instead...")?;
    
    // Compare with main
    let diff = db.diff("main", "try-microservices")?;
    
    Ok(())
}
```

## How It Compares

| Feature | Indra | Other Memory Systems |
|---------|-------|---------------------|
| **Branching** | ✅ Full git-like branches | ❌ Linear only |
| **Diff/Compare** | ✅ Any two points | ❌ Not available |
| **History** | ✅ Full commit history | ❌ Current state only |
| **Semantic Search** | ✅ Local HF models | Varies |
| **Storage** | ✅ Single portable file | Multi-file |
| **Visualization** | ✅ 3D via IndraDB | ❌ Not available |

## Architecture

```
.indra (single file)
├── Header
├── Objects (content-addressed, zstd compressed)
│   ├── Entries (content + embedding)
│   ├── Commits (snapshot + parents + message)
│   └── Trees (merkle trie for structural sharing)
├── Index (hash → offset)
└── Refs (branch → commit)
```

**Key design choices:**
- **BLAKE3** hashing (fast, secure)
- **Merkle trie** for efficient branching (structural sharing)
- **Embeddings stored with content** (deduplicated)
- **Single file** for portability

## Visualize with IndraDB

Push to [IndraDB](https://indradb.net) to see your agent's reasoning in 3D:

```bash
indra login
indra remote add origin username/my-agent
indra push origin
```

## License

MIT

## Etymology

Named after [Indra's net](https://en.wikipedia.org/wiki/Indra%27s_net) — a Buddhist metaphor where reality is a net of jewels, each reflecting all others. Your reasoning forms a similar web of interconnected insights.
