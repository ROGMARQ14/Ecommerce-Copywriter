# Ecom Collection Copywriter

## Overview
- SEO copywriter for ecommerce collection pages across multiple agency clients.
- Target audience: internal use only (SEOptimize agency).
- Communication mode: **Concise**.
- Markdown-only workflow — drafts are handed off to clients for CMS implementation.

## Source of Truth

**`.claude/skills/collection-page-draft/SKILL.md` is the authoritative spec** for how to draft, structure, and QA a collection page. It defines the 3-step workflow, the locked page structure, keyword placement rules, voice rules, banned phrases, and the no-invented-facts rule. Every command and agent in this project reads from it. If anything in this CLAUDE.md conflicts with the skill, the skill wins.

## Tech Stack
- Markdown files only. No build step, no runtime.
- **WebFetch** for browsing live collection URLs (Step 1 of the workflow).
- **Context7 MCP** for Shopify Liquid, schema.org, ecom platform docs lookup.

## Project Structure
```text
ecom-collection-copywriter/
├── CLAUDE.md
├── CLAUDE.local.md          # personal overrides (gitignored)
├── .env.example
├── .gitignore
├── 04-collection-page-prompt-final.md  # original prompt spec, source for SKILL.md
├── .claude/
│   ├── settings.json
│   ├── commands/new-collection.md
│   ├── skills/collection-page-draft/SKILL.md   # ← source of truth
│   └── agents/
│       ├── brief-strategist.md    # pre-draft strategy
│       ├── copy-drafter.md        # Step 3 drafter
│       └── draft-qa.md            # post-draft QA
├── clients/
│   └── {client-slug}/
│       ├── brand-voice.md       # tone, style, banned words, voice rules
│       ├── product-catalog.md   # known SKUs/specs to prevent invention
│       └── collections/
│           └── {collection-slug}.md  # generated drafts
└── docs/decisions/          # ADRs
```

## Commands
- **/new-collection** — Orchestrates the 3-step workflow: WebFetch the URL → propose heading structure with a hard pause for user approval → write the full locked-structure draft after approval. Saves to `clients/{client}/collections/{slug}.md`.

## Agents
- **brief-strategist** — Optional pre-draft strategist. Produces a structured brief with H2/H3 angles, FAQ topics, semantic coverage plan, voice notes. Spawn when intake is complex.
- **copy-drafter** — Executes Step 3 drafting from an approved heading set.
- **draft-qa** — Post-draft QA. Flags structural, keyword, voice, and factual issues against the skill's checklist. Does not rewrite.

## The 3-Step Workflow (non-negotiable)
1. **Step 1** — WebFetch the URL. Read product listings. Silently build the semantic field from PRIMARY KW 1, PRIMARY KW 2, and SECONDARY KWs. No output.
2. **Step 2** — Propose H1 + H2 + H3.1 + H3.2 + H3.3. **Hard stop.** End the turn. Wait for user approval before any drafting.
3. **Step 3** — Write the full locked-structure page once headings are approved.

## Trigger Message Format
When running `/new-collection`, users supply (as message 2, or when prompted):

```text
URL: [full collection page URL]
PRIMARY KW 1: [priority primary keyword]
PRIMARY KW 2: [secondary primary keyword]
SECONDARY KWs: [comma-separated long-tail and modifier keywords]
Status: NEW or EXISTING
Metadata (EXISTING only): [paste current URL, title, meta title, meta description]
Existing copy (EXISTING only): [paste current page copy]
```

## Locked Page Structure (high level — full spec in skill)
1. **Metadata block** — NEW (new URL/title/meta) or EXISTING (keep originals + Revised Meta Title + Revised Meta Description)
2. **H1** — exact match PRIMARY KW 1, under 60 chars, no brand name
3. **Short Description 1** — 60 words target, 75 word hard cap, 3-sentence formula (opener with PRIMARY KW 1 in first 10 words → human moment → grounding detail)
4. **`{product grid}`** — on its own line
5. **H2** — under 70 chars, contains PRIMARY KW 2 or SECONDARY variant, decision/benefit angle
6. **3× H3 with descriptions** — 80-120 words each, 300-400 total; each H3 covers a different dimension (format, persona, use case, quality, running cost, care, etc.); each description ends with a human last sentence
7. **Shop similar collections** — `Shop similar collections: [Anchor 1] | ... | [Anchor 5]` (4-6 anchors, each an exact primary keyword of destination, never self-link)
8. **FAQ** — 4-5 decision-stage questions, 1-2 sentence direct answers, top two most keyword-rich
9. **Status Summary** — `Page: [name] | Primary KW 1 uses: [N] | Primary KW 2 uses: [N] | Word count: [N]`

Total body word count: 400-600.

## Critical Rules
- **No invented facts.** Never fabricate specs, materials, dimensions, claims, prices, offers, certifications, or shipping/return policies. Every product-specific claim must trace to the live URL, `product-catalog.md`, or the supplied brief. If details aren't available, write category-level copy or use `[CONFIRM WITH CLIENT]`.
- **Approval gate is mandatory.** Step 2 ends the turn. Never proceed to Step 3 without user approval. Never use `AskUserQuestion` for the approval — this is a hard pause, not a question.
- **Keyword placement rules are exact.** PRIMARY KW 1 in H1 (exact), first 10 words of Short Desc 1 (with natural opener, never bare), at least one H3. PRIMARY KW 2 in Short Desc 1 sentence 3 if it fits, in H2 or one H3. Never same sentence as KW 1. SECONDARY KWs never inserted.
- **Word "collection"** must not appear anywhere on the page.
- **No em dashes** in copy.
- **No 3+ comma-linked clauses.** Comma-heavy sentences are the clearest AI tell.
- **Banned phrases list** (full list in skill) — rewrite the whole sentence if any appear.
- **Brand voice fidelity.** If `clients/{client}/brand-voice.md` exists, load and follow it. Otherwise apply generic ecom voice and flag it.
- **Platform-formatting agnostic.** Plain markdown only. No Shopify Liquid, WooCommerce shortcodes, or platform-specific syntax unless explicitly requested.
- **Compliance.** For regulated categories (supplements, CBD, alcohol, financial, medical), soften efficacy/health claims and add FTC reminder in Notes for Client.

## MCP Servers
- **Context7** — Shopify Liquid, schema.org CollectionPage/ItemList, or ecom platform docs.

## Current Phase
- **Phase**: Locked structure + 3-step workflow in place, mirroring `04-collection-page-prompt-final.md`.
- **Next**: First run of the new `/new-collection` against a real client brief.
- **Decisions**: Markdown-only, single command (`/new-collection`), skill is source of truth, 3 agents (brief-strategist, copy-drafter, draft-qa), hard approval gate after heading proposal.
