# Tech Brain

A personal, Git-backed technical knowledge base for turning things I read and learn into concise, connected notes I can revisit later.

## Structure

- `inbox/` — articles, links, transcripts, and rough material waiting to be processed.
- `notes/` — durable notes written for quick recollection.
- `index.md` — navigation, topic summaries, and unresolved knowledge gaps.

`AGENTS.md` defines how AI agents compile, connect, and maintain the notes. `CLAUDE.md` points Claude to the same instructions.

## Workflow

1. Put new material in `inbox/`.
2. Ask an agent to process it using `AGENTS.md`.
3. Review the resulting note changes and connections.
4. Sync through Obsidian Git or normal Git commands.

Example prompt:

> Process `inbox/<file>` using `AGENTS.md`. Integrate the durable ideas into existing notes, preserve citations, connect related topics, and update `index.md`.

To save learning from a chat:

> Compile the durable technical learnings from this conversation into the vault using `AGENTS.md`. Prefer updating existing notes, preserve source links, and show me what changed.

## Open in Obsidian

Choose **Open folder as vault** and select this repository. The included Obsidian Git configuration pulls on startup, checks for remote changes every 15 minutes, and commit-syncs every 60 minutes.

Do not store credentials, private data, or employer-confidential material in the vault.
