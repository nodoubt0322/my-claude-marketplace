---
name: generating-progress-tracker
description: Use when the user has a PRD, plan, spec, or roadmap document and needs a progress.md to track phase-based execution status, when finishing writing a plan/PRD and needing a companion tracker, when checking off completed work during implementation, or when transitioning between phases of multi-stage work
---

# Generating Progress Tracker

## Overview

Produce and maintain a `progress.md` companion file for a PRD/plan/spec. The file uses a strict phase-based checkbox format so it can be machine-read and human-scanned. This skill defines **the format**, **the state vocabulary**, **the merge protocol** for existing files, and **the auto-update protocol** during execution.

**Core principle:** `progress.md` is a faithful mirror of the source plan, never an embellishment of it. Do not add commentary, summaries, "overall progress" sections, or invented criteria. If it is not in the source, do not put it in the tracker.

## When to Use

Trigger this skill in any of these situations:

1. **Explicit request** — user asks for "進度表 / progress tracker / progress.md" or invokes `/progress`.
2. **After writing a plan/PRD** — when you finish authoring a plan document, proactively offer to generate the matching `progress.md`.
3. **During implementation** — when you complete a todo that exists in `progress.md`, update its checkbox (after announcing the change — see Auto-Update Protocol).
4. **Phase transition** — before moving from phase N to phase N+1, review the previous phase's verification checkboxes.

## The Format (STRICT)

```markdown
# Progress: <project / feature name>

> Source: <relative or absolute path to the PRD/plan file>
> Last updated: <YYYY-MM-DD>

## Phase 1: <title>

- [ ] todo 1
- [ ] todo 2
- [ ] todo 3

---

- [ ] 驗證重點 1
- [ ] 驗證重點 2

## Phase 2: <title>

- [ ] todo 1
- [ ] todo 2

---

- [ ] 驗證重點 1
```

**Format rules — DO NOT deviate:**

- Use `## Phase N: <title>` exactly (English "Phase" + arabic number, even if source uses 「階段一」/「Step 1」 — normalize to `Phase N` and keep the original title text).
- A single `---` (three dashes) line separates todos from verification points within one phase.
- **No subheadings** inside a phase. No `### Tasks`, no `### 驗證重點`, no `**目標**`, no description paragraphs.
- **No "Checkbox 圖例" block.** State legend is documented in this skill, not in the output file.
- **No "總體進度 / Overall Progress" summary block.** Phases are the only structure.
- Header includes `Source:` and `Last updated:` lines only — nothing else.

## Checkbox State Vocabulary (6 states)

| Symbol | Meaning | When to use |
|--------|---------|-------------|
| `[ ]` | 未開始 | Default initial state |
| `[~]` | 進行中 | Work has started but not done |
| `[x]` | 完成 | Done **and** verified |
| `[!]` | 阻塞/有問題 | Stuck; append ` — <reason>` after the item |
| `[?]` | 待確認 | Needs clarification before progress; append ` — <question>` |
| `[-]` | 已取消 | Decided not to do; keep the line so history is visible |

**Critical:**
- A todo only becomes `[x]` when verified (tests pass / behavior confirmed / criterion met). "I wrote the code" is not `[x]`; it is `[~]`.
- `[!]` and `[?]` **must** carry an inline annotation explaining the blocker or question.
- Never delete a `[-]` item — its presence records the decision.

## Generation Workflow

1. **Read the source document fully.** Identify phase boundaries. A "phase" may be called: `階段`, `Phase`, `Step`, `Milestone`, `Sprint`, `Part`, numbered sections, or a clearly grouped block. Normalize all to `Phase N`.
2. **Extract todos verbatim.** Use the wording in the source. Do not paraphrase or rewrite.
3. **Split rule:** Split a sentence into multiple todos **only** when the source uses an explicit enumeration: a bulleted/numbered list, an Oxford-comma list of parallel verbs/objects (`do A, do B, and do C`), or a 頓號/逗號 list of parallel nouns (`session store、dependencies、設定`). Do **not** split a single descriptive sentence into "verb + object" pieces. When in doubt, keep it as one item.
4. **Extract verification points.** A line goes below `---` when it describes an **observable pass/fail check on the result** (test passes, build green, lint clean, URL works, coverage > N%, review approved). A line stays as a todo when it describes **work to perform** (write tests, set up config, refactor X). The same noun can appear in both: "撰寫單元測試" is a todo; "單元測試全綠" is a verification.
5. **Mixed sentences** (`do X and ensure Y`): split on the conjunction — the "do X" half is a todo, the "ensure Y" half is a verification.
6. **If a phase has no verification in the source**, write a single line: `- [ ] (待補充驗收條件)` — do not invent criteria.
7. **Place the file next to the source** by default: same directory as the PRD/plan, named `progress.md`. If multiple progress files would collide, use `<source-stem>-progress.md`.
8. **Set `Last updated` to today's date** (YYYY-MM-DD).

## Merge Protocol (when progress.md already exists)

**Never blindly overwrite.** Run this protocol:

1. Read the existing `progress.md`.
2. Re-extract structure from the (possibly updated) source.
3. For each todo/verification item in the new structure:
   - If a matching line (same wording) exists in the old file → **keep its current checkbox state** (including `[x]`, `[!]`, `[?]`, `[-]`, and any inline annotations).
   - If it is new → insert with `[ ]`.
4. For each item in the old file that is **not** in the new structure:
   - If it was `[x]` or `[-]` → keep it under its phase, mark with a trailing ` — (removed from plan)` annotation so history is preserved.
   - If it was `[ ]` → drop it.
   - If it was `[~]`, `[!]`, or `[?]` → keep it and flag for the user with a trailing ` — (no longer in plan; confirm?)`.
5. Update `Last updated:` to today.
6. **Report the diff** to the user before saving: "Added N items, removed M, kept Z, flagged F for review."

## Auto-Update Protocol (during execution)

When you (Claude) complete or change the state of work that maps to an item in `progress.md`:

1. **Announce first.** Say: "我要把 `<phase> / <todo>` 標為 `[x]` / `[~]` / `[!]`" before editing.
2. **Edit the single line.** Do not reformat the rest of the file.
3. **Update `Last updated:`.**
4. **Only mark `[x]` after verification.** If you only finished implementation but have not run the test/check, use `[~]`.

## Phase Transition Protocol

Before starting Phase N+1:

1. Re-read Phase N's verification section.
2. For each verification item: explicitly confirm `[x]` (with evidence) or surface why it is not yet `[x]`.
3. If any verification is still `[ ]`, ask the user whether to proceed or finish verification first.

## Worked Examples for the Split/Verification Rules

**Example A — `do X and ensure Y` (split on conjunction):**
> Source: "需要寫一份 design doc 並讓 tech lead review"
- todo: `- [ ] 寫一份 design doc`
- verification: `- [ ] tech lead review 通過`

**Example B — write test ≠ test passes:**
> Source: "撰寫對應的單元測試"
- todo: `- [ ] 撰寫對應的單元測試`
- verification: nothing (do not invent "測試全綠"). If the phase truly has no other verification, write `- [ ] (待補充驗收條件)`.

**Example C — 頓號 list of parallel nouns (split):**
> Source: "刪除舊的 session store、相關 dependencies、設定"
- todo: `- [ ] 刪除舊的 session store`
- todo: `- [ ] 刪除相關 dependencies`
- todo: `- [ ] 刪除相關設定`

**Example D — single descriptive sentence (do NOT split):**
> Source: "POST /api/todos 新增任務"
- todo: `- [ ] POST /api/todos 新增任務` (one line; do not split into "建立 endpoint" + "處理新增邏輯")

## Quick Reference

| Situation | Action |
|-----------|--------|
| Source uses `階段` / `Step` / `Milestone` | Normalize to `Phase N`, keep original title text |
| Sentence has explicit list (頓號 / and / `,`) | Split into parallel todos |
| Single descriptive sentence | Keep as one todo |
| `do X and ensure Y` | Split: X is todo, Y is verification |
| "撰寫測試" in source | Todo only; do not invent "測試全綠" verification |
| No verification in source | Write `- [ ] (待補充驗收條件)` |
| File exists | Run Merge Protocol, never overwrite |
| Completed a todo | Announce → edit → update `Last updated` |
| Implementation done, tests not run | Use `[~]`, not `[x]` |
| Stuck on a todo | `[!] — <reason>` |
| Need user input on a todo | `[?] — <question>` |
| Plan dropped a todo that was done | Keep with `— (removed from plan)` |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Adding "Checkbox 圖例" / state legend to the file | Remove it. The legend is in this skill, not the output. |
| Adding "總體進度 / Overall Progress" summary | Remove it. Phases are the only structure. |
| Adding `**目標**` / `### Tasks` / `### 驗證重點` subheadings | Remove them. Use the strict format only. |
| Marking `[x]` because code is written | Downgrade to `[~]` until verification passes. |
| Inventing acceptance criteria | Use `- [ ] (待補充驗收條件)` instead. |
| Overwriting existing file | Run Merge Protocol. |
| Paraphrasing source todos to "improve" them | Copy verbatim. |
| Translating `階段一` to `Phase 1` and losing the original title text | Normalize the prefix only; preserve the original title after the colon. |
| Updating checkboxes silently during work | Announce first, then edit. |

## Red Flags — STOP and re-read this skill

- About to add a section the format does not show
- About to write `[x]` without having verified
- About to overwrite an existing `progress.md`
- About to invent verification criteria the source did not contain
- About to use Chinese phase prefix (`階段一`) instead of `Phase 1`
- About to update a checkbox without announcing
