# book-to-skill-distillation

## ⚡ Quick Start

```bash
git clone https://github.com/9s5bz2jvd2-lang/book-to-skill-distillation
cd book-to-skill-distillation
pip install pymupdf
python scripts/setup_distill.py your-book.pdf
# → Workspace created. Now follow SKILL.md for distillation.
```

**What this does**: Takes a book/PDF and converts it into an agent-native skill — routers, checklists, decision trees, not a summary.

---

## What is this?

This is a **loop engineering** skill for [LingTai](https://github.com/Lingtai-AI/lingtai) agents. It converts books, long PDFs, EPUBs, manuals, guideline collections, and databases into **agent-native skill structures** — the branching, callable artifacts an agent uses at runtime.

> *"You shouldn't be prompting coding agents anymore. You should be designing loops that prompt your agents."* — Peter Steinberger

This skill IS a loop. You don't prompt the agent to "read chapter 3 and take notes." You design a system that:
1. **Finds** relevant material via sparse routing (`ROUTING.yaml`)
2. **Hands out** work to parallel workers (daemon fan-out)
3. **Checks** quality via missed-case sweep
4. **Records** what's done in structured stores (`source_catalog.jsonl`, `node_store.jsonl`)
5. **Decides** the next step based on graph edges and budget tiers
6. **Returns** compressed artifacts to cache for future calls

## Why not just summarize?

Most "book → notes" tools produce shorter prose. That's the wrong shape for an LLM agent. A linear chapter summary doesn't help an agent:
- **Route** a task to the right procedure
- **Check** whether it missed an edge case
- **Decide** whether it needs to load a heavy reference
- **Verify** whether a claim depends on current law or pricing

This skill encodes a repeatable workflow for the *rewriting* step: taking linear human text and turning it into the branching, callable artifacts an agent uses at runtime.

## How it works — the Loop

```
                    ┌─────────────────────────────────────────────┐
                    │                                             │
                    ▼                                             │
           ┌──────────────┐                                      │
           │  Shared Core  │  ← Always loaded first (200-800 tok) │
           │  (copyright,  │                                      │
           │   safety,      │                                     │
           │   red lines)   │                                     │
           └──────┬───────┘                                      │
                  │                                               │
                  ▼                                               │
        ┌─────────────────┐        ┌──────────────────┐          │
        │  ROUTING.yaml   │ ─────▶ │  Top-k Experts   │          │
        │  (sparse        │        │  (800-3000 tok   │          │
        │   activation)   │        │   each)          │          │
        └────────┬────────┘        └────────┬─────────┘          │
                 │                          │                     │
                 ▼                          ▼                     │
        ┌─────────────────┐        ┌──────────────────┐          │
        │  Missed-case    │        │  Route Log       │          │
        │  Sweep          │◀──────│  (what was       │          │
        │  (systematic    │        │   loaded/skipped) │          │
        │   edge-case     │        └──────────────────┘          │
        │   checklist)    │                                      │
        └────────┬────────┘                                      │
                 │                                                │
                 ▼                                               │
        ┌─────────────────────────────────┐                     │
        │  Cyclic Return-to-Cache         │─────────────────────┘
        │  (compress results back into    │   (better cache hit
        │   ROUTING, GRAPH, CACHE, evals)  │    next time)
        └─────────────────────────────────┘
```

### Key concepts

| Concept | What it means | Where it lives |
|---------|--------------|----------------|
| **Sparse activation** | Load only what the task needs — not the whole skill | `ROUTING.yaml` |
| **Shared core** | Always-on safety/copyright/red-line rules (≤800 tokens) | `SKILL.md` |
| **Top-k routed experts** | The smallest sufficient expert set for the task | `reference/` |
| **Missed-case sweep** | Systematic edge-case checklist after sparse activation | `ROUTING.yaml` |
| **Budget tiers** | Token budget per component type | `ROUTING.yaml` |
| **Cyclic return-to-cache** | Compress results back into routing/cache for next time | `CACHE.md` |
| **Source packs** | Multi-book domain routing with conflict gates | `ROUTING.yaml` |

## What's in this repo

```
SKILL.md                     # Thin sparse router — first file an agent loads
RUNBOOK.md                   # Demand-sensitive sparse/full reading execution manual
ROUTING.yaml                 # Machine-readable activation hints, budget tiers, triggers
CACHE.md                     # Cache-friendly layout design (stable prefix + variable suffix)
RULES.md                     # Ownership, reuse boundaries, content safety

scripts/
  setup_distill.py           # Bootstrap: PDF → workspace (TOC + fulltext + catalog)
  route_plan.py              # Generate demand-sensitive sparse reading plan
  fanout_daemons.py          # Generate parallel daemon tasks from a doc list
  quality_check.py           # Lightweight release checks
  extract_bookmarks.py       # Dump PDF TOC as JSON
  scout.sh                   # Quick PDF reconnaissance (pdfinfo + sample text)
  render_pages.sh            # Render PDF pages to PNG for OCR

reference/
  sparse-distillation-methodology.md   # Full methodology
  toc-recovery.md                      # TOC extraction patterns
  validation-checklist.md              # Output validation
  worked-example-legal-dd.md           # Worked example

assets/
  output-template.md                   # Distilled skill module template
  route-log-template.md                # Route log template
  eval-cases.md                        # Evaluation cases
  route-log-examples.md                # Route log examples
  reference-doc-template.md            # Reference doc template

examples/
  quickstart-example.md                # Concrete before/after worked example
```

## Operational discipline

- **Copyright-safe**: Extract operational know-how (procedures, checklists, taxonomies); do not reproduce prose or examples beyond tiny necessary snippets.
- **Privacy-safe**: Source-derived OCR substrate stays under a private `work/book-distill/<slug>/` workspace, not inside the published skill.
- **Verification-aware**: Any claim depending on current law, pricing, or API behavior is marked for refresh against an authoritative source.
- **Token-aware**: Budget tiers in `ROUTING.yaml` control token consumption per component type. The loop is designed to be token-efficient by default.

## Multi-book domain support

For same-domain multi-book distillation (e.g., a nutrition guideline library, a legal handbook collection):

1. Each book becomes a **source pack** with its own `source_map`
2. A **domain parent skill** routes over source packs
3. **Source anchors** preserve which book said what
4. **Conflict gates** log contradictions between sources
5. **Unified graph index** connects nodes across all books

Rule: *Do not blend sources into anonymous consensus; preserve source identity and log conflicts.*

## Relation to 生万物论 (Sheng Wanwu)

This skill is the **成库 (library) layer** for [生万物论](https://github.com/9s5bz2jvd2-lang/shengwanwu) — a hypothesis discovery engine that builds on distilled knowledge bases:

```
道 (papers) → 一 (distill to library) → 二 (graph) → 三 (hypothesize) → 万物 (discover)
```

`book-to-skill-distillation` handles the 道→一→二 layers. `shengwanwu` handles the 二→三→万物 layers.

## License

© 2026 王润圆 (Wang Runyuan) · Kunming Medical University · Master of Nutrition and Food Hygiene

- ✅ Allowed: Share, adapt, learn, research
- ❌ Prohibited: Any form of commercial use
- Attribution required when referencing

## Author

**王润圆 (Wang Runyuan)**
Kunming Medical University · Master of Nutrition and Food Hygiene
China Registered Dietitian

---

*Powered by [LingTai](https://github.com/Lingtai-AI/lingtai) — agent-native loop engineering.*

---

## 📜 许可 · License

本项目为公益开源，采用 **MIT 许可证**：

- ✅ **随意使用**：学习、研究、转载、二次创作、**商用也可以** —— 保留原作者署名（王润圆 Runyuan Wang）就好啦 💛
- 🌱 开源是为了帮助更多的人。

This project is public-welfare open source under the **MIT License** — free for anything, **including commercial use**; just keep the original credit ("Runyuan Wang") 💛

© 2026 王润圆 Runyuan Wang · MIT License
