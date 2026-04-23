# ADR 0001 — Markdown-Only Workflow

**Date:** 2026-04-22
**Status:** Accepted

## Context
The agency needs a repeatable copywriter for ecommerce collection pages across multiple clients. Options ranged from full automation (n8n + Supabase + ecom API push) to plain markdown drafts handed off to clients.

## Decision
Markdown-only. Drafts live in `clients/{client}/collections/{slug}.md`. Clients implement on their own CMS.

## Rationale
- No infra to maintain.
- Drafts are easily reviewed, versioned, edited, and shared.
- Each client uses a different ecom platform — emitting platform-specific syntax would require per-client adapters.
- Future automation layers (Shopify push, GSC pull) can sit on top of the markdown without changing the writing workflow.

## Consequences
- Clients handle CMS implementation themselves.
- No automatic publish path. Acceptable trade-off for v1.
- If the agency later wants direct publish, add a per-platform exporter without changing the draft format.
