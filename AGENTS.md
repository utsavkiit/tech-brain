# Personal Tech Knowledge Base Instructions

This repository is a simple Obsidian wiki. It has only three working surfaces:

- `inbox/` for unprocessed material.
- `notes/` for all durable knowledge.
- `index.md` for navigation and open gaps.

## Before editing

1. Read `index.md` and search `notes/` for related ideas.
2. Prefer improving an existing note over creating a near-duplicate.
3. Preserve unrelated user edits.
4. Name new note files in `kebab-case.md`, matching the note's H1 title, so filenames stay consistent and near-duplicates are easy to spot at a glance.

## Compile an inbox item

1. Read the item completely and identify the durable claims.
2. Update the relevant notes or create a clearly named note when the idea is genuinely distinct.
3. Write for a human returning months or years later. Lead with a concise explanation that restores the overall idea without requiring the original source.
4. Write synthesis in your own words; do not copy long passages.
5. Put citations and URLs in a `## Sources` section inside each affected note.
6. Distinguish source claims, inference, and personal experience.
7. Add meaningful Obsidian links and explain non-obvious relationships in prose.
8. Update `index.md` with new notes and unresolved gaps or contradictions.
9. Remove the inbox item only after its useful content and provenance are preserved.

## Write for recollection

Each note should help the user quickly reconstruct the idea, not merely archive facts. Adapt the structure to the topic, but normally include:

- **In brief:** a few sentences explaining the core idea in plain language.
- **Why it matters:** the problem it addresses and when it becomes useful.
- **How it works:** the essential mechanism or reasoning, including prerequisites.
- **Example:** a concrete scenario, analogy, or small technical example when helpful.
- **Tradeoffs and boundaries:** where the idea fails, what it costs, and what it is commonly confused with.
- **Connections:** how it relates to existing notes and why those relationships matter.
- **Sources:** enough context to revisit the original material.

Put the most recall-worthy information near the top. Define unfamiliar terminology on first use, preserve important nuance, and favor clear explanations over exhaustive detail. Do not include a section merely to satisfy a template when it adds no value.

## Keep notes concise

- State each idea once, in the section where it is most useful. Do not repeat the introduction in the conclusion or restate the same claim across multiple sections.
- Remove generic framing, padded transitions, motivational commentary, obvious observations, and formulaic AI language.
- Prefer a short paragraph or a few strong bullets over exhaustive lists.
- Include examples, caveats, and implementation detail only when they materially improve understanding or prevent a likely misunderstanding.
- Link to an existing note instead of re-explaining its subject. Include only the context needed to understand the connection.
- Preserve useful nuance, but move tangents and distinct concepts into their own notes only when they have durable value.
- Before finishing, perform an editing pass that removes redundancy and compresses wording without losing meaning.

The target is the shortest note that lets the user accurately recover the idea—not the most comprehensive note the agent can generate.

## Synthesize and complete the graph

When asked to synthesize or find missing links, compare related notes for agreements, tensions, prerequisites, consequences, and unanswered questions. Strengthen canonical notes, add useful reciprocal connections, and record unresolved gaps in `index.md`. Do not create a new summary note unless it represents a durable concept.

## Quality bar

A useful note should stand on its own well enough that the user can recover the mental model and explain the idea after a quick reread. It explains what the idea is, why it matters, how it works, its tradeoffs and boundaries, its connections, and its evidence—with no unnecessary repetition. Avoid contextless bullet dumps, generic filler, fabricated citations, duplicate notes, bloated summaries, and links added only to make the graph look dense.
