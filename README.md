# Ecom Collection Copywriter

SEO copywriter for ecommerce collection pages. Uses Claude Code with a locked 3-step workflow (browse → propose headings → write full page) to produce SEO-optimized, brand-voice-accurate collection page drafts that never invent product facts.

Runs on markdown files only. No build step, no runtime.

---

## What this repo does

Give it:
- A live collection page URL
- Two primary keywords + a list of secondary keywords
- A client (brand voice + product catalog, if available)

Get back:
- A full collection page draft: metadata, H1, short description, product grid, H2 + 3× H3 buyer's guide, "shop similar collections" links, FAQ, and a status summary.
- Saved to `clients/{client}/collections/{slug}.md`.

The full spec lives in [`.claude/skills/collection-page-draft/SKILL.md`](.claude/skills/collection-page-draft/SKILL.md) — that file is the source of truth for the workflow and output.

---

## Setup

### 1. Install Claude Code
Get it from [claude.com/claude-code](https://claude.com/claude-code). Works in terminal, desktop app, or the VS Code extension.

### 2. Clone this repo
```bash
git clone <repo-url>
cd ecom-collection-copywriter
```

### 3. Create your local config (optional)
Copy `CLAUDE.local.md` pattern for personal overrides (gitignored):
```bash
echo "# Personal overrides" > CLAUDE.local.md
echo "# Your email: you@agency.com" >> CLAUDE.local.md
echo "# Today's date: 2026-XX-XX" >> CLAUDE.local.md
```

### 4. Open the project in Claude Code
Claude Code will auto-detect `.claude/` and load the command, agents, and skill. No other install needed.

### 5. (Optional) Copy `.env.example` to `.env`
Only needed if you extend the project with API integrations. The default workflow uses WebFetch and Context7 MCP only, both of which work without keys.

---

## Usage

### Quick start: draft a new collection page

In Claude Code, type:
```
/new-collection
```

Then paste the trigger message in this format:
```text
URL: https://example.com/collections/under-sink-water-filter
PRIMARY KW 1: under sink water filter
PRIMARY KW 2: under bench water filter
SECONDARY KWs: affordable under sink water filters, best under sink water filter Australia, under bench water filter with faucet
Status: NEW
```

For existing pages, set `Status: EXISTING` and paste the current metadata + copy.

Claude will:
1. **Step 1**: Browse the URL, read the product listings, silently build a semantic field from the keywords.
2. **Step 2**: Propose H1, H2, and 3× H3 headings. **Hard stop** — you approve or request changes before any copy is written.
3. **Step 3**: Write the full draft in the locked structure, QA it against the checklist, save to `clients/{client}/collections/{slug}.md`, and report.

### Other commands and agents

- **brief-strategist** agent — spawn before `/new-collection` when you want A/B heading options or a structured brief first.
- **copy-drafter** agent — handles Step 3 drafting in isolation.
- **draft-qa** agent — reviews a finished draft against all quality gates, returns a pass/fail report. Use before handing work to a client.

---

## Adding a new client

1. Copy the scaffold:
   ```bash
   cp -r clients/_template clients/{new-client-slug}
   ```
2. Fill in `clients/{new-client-slug}/brand-voice.md` with the client's tone, banned words, voice rules.
3. Fill in `clients/{new-client-slug}/product-catalog.md` with verifiable product specs/SKUs (used to prevent invented facts).
4. Collection drafts will auto-save to `clients/{new-client-slug}/collections/` when you run `/new-collection`.

By default, real client folders are **gitignored** (see `.gitignore`). Only `clients/_template/` is committed. To share a specific client's work in the repo, add `!clients/{client-slug}/` to `.gitignore` explicitly.

---

## Project structure

```text
ecom-collection-copywriter/
├── CLAUDE.md                    # project-wide rules, points to the skill
├── CLAUDE.local.md              # personal overrides (gitignored)
├── README.md                    # this file
├── .gitignore
├── .env.example
├── 04-collection-page-prompt-final.md   # original spec, preserved for reference
├── .claude/
│   ├── settings.json            # tool allowlist (WebFetch, Context7 MCP, etc.)
│   ├── commands/
│   │   └── new-collection.md    # /new-collection orchestrator
│   ├── agents/
│   │   ├── brief-strategist.md  # pre-draft strategist
│   │   ├── copy-drafter.md      # Step 3 drafter
│   │   └── draft-qa.md          # post-draft QA
│   └── skills/
│       └── collection-page-draft/
│           └── SKILL.md         # ← source of truth for everything
├── clients/
│   ├── _template/               # scaffold for new clients (committed)
│   │   ├── brand-voice.md
│   │   ├── product-catalog.md
│   │   └── collections/
│   └── {client-slug}/           # real client work (gitignored by default)
└── docs/
    └── decisions/               # ADRs for project decisions
```

---

## The locked output structure

Every draft follows this exact order (full spec in the skill):

1. **Metadata block** — NEW (new URL + title + meta) or EXISTING (keep originals + Revised versions)
2. **H1** — exact match PRIMARY KW 1, under 60 chars
3. **Short Description 1** — 60 words target (75 cap), 3-sentence formula
4. **`{product grid}`** — on its own line
5. **H2** — contains PRIMARY KW 2 or a secondary variant, decision/benefit angle
6. **3× H3 + descriptions** — 80–120 words each, each covers a different buyer dimension
7. **Shop similar collections** — 4–6 related page anchors
8. **FAQ** — 4–5 decision-stage questions, direct 1–2 sentence answers
9. **Status Summary** — keyword usage counts + word count

Total body copy: 400–600 words.

---

## The rules that matter

Full ruleset in the skill. The ones that are most often violated:

- **No invented facts.** Every product-specific claim must trace to the live URL, the product catalog, or the supplied brief.
- **Keyword placement is exact.** PRIMARY KW 1 in H1, first 10 words of Short Desc 1 (with a natural opener, never bare), and at least one H3. PRIMARY KW 2 in Short Desc 1 sentence 3, H2 or one H3. Never in the same sentence as KW 1.
- **Never use the word "collection"** on the page.
- **No em dashes** in copy. No 3+ comma-linked clauses.
- **Banned phrases list** — rewrite the whole sentence if any appear (full list in the skill).
- **Approval gate is mandatory.** Step 2 ends the turn. No drafting until you approve headings.

---

## Troubleshooting

**The workflow skipped Step 2's approval pause.**
The command is strict about this. If Claude drafted without approval, interrupt and restart. Step 2 must end the turn.

**WebFetch failed on the URL.**
The workflow notes this at the top of the output and proceeds from the pasted copy. Make sure you included the existing page content in the trigger for `Status: EXISTING` pages.

**Draft has invented facts.**
Run the `draft-qa` agent — it flags every claim that doesn't trace to the catalog or brief. Add missing facts to `clients/{client}/product-catalog.md` and re-draft.

**Claude is using the wrong voice.**
Make sure `clients/{client}/brand-voice.md` exists and is filled in. The drafter loads it automatically if present.

---

## Who this is for

Internal use at SEOptimize. Communication mode is Concise — the drafter and agents don't preamble or summarize unnecessarily. Output is plain markdown, handed off to clients for CMS implementation.
