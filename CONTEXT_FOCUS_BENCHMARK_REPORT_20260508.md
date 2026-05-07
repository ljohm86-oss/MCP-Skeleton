# Context Focus Benchmark Report

This report summarizes one formal `focus-mode` benchmark run from the standalone `MCP-Skeleton` repository.

The goal is simple:

- quantify whether focus-mode skeletons are smaller than the default `full` view
- separate code-directory results from long-text results
- show where focus modes improve readability only versus where they also reduce skeleton size

## Benchmark Scope

This report is based on the quick benchmark fixture from:

- `/Users/carwynmac/MCP-Skeleton/testing/context_scale_benchmark.py`

Environment:

- generated_at: `2026-05-07T16:48:00Z`
- repo_root: `/Users/carwynmac/MCP-Skeleton`
- python: `3.9.6`
- platform: `macOS-26.4.1-arm64-arm-64bit`

Important boundary:

- this is a **controlled quick fixture benchmark**
- it is useful for comparing focus modes against each other
- it is **not** a substitute for a larger repo-scale benchmark on a real production repository

## Focus Modes Under Test

Directory-oriented:

- `full`
- `tree`
- `imports`
- `symbols`

Text-oriented:

- `full`
- `writing-outline`

## Directory Focus Results

The directory fixture showed a clear and consistent pattern:

- `tree`
- `imports`
- `symbols`

all reduced the skeleton surface to roughly half of the default `full` view.

### Heuristic backend

| Focus mode | Full skeleton chars | Focused skeleton chars | Char ratio | Full skeleton tokens | Focused skeleton tokens | Token ratio | Compress ratio |
| --- | --- | --- | --- | --- | --- | --- | --- |
| tree | 1,871 | 903 | 0.4826 | 468 | 226 | 0.4829 | 0.9907 |
| imports | 1,871 | 938 | 0.5013 | 468 | 235 | 0.5021 | 0.9766 |
| symbols | 1,871 | 920 | 0.4917 | 468 | 230 | 0.4915 | 0.9715 |

### Tokenizer-backed backends (`auto` / `tiktoken`)

The same pattern held under tokenizer-backed counting:

- `tree` reduced skeleton tokens to about `48.25%`
- `imports` reduced skeleton tokens to about `48.91%`
- `symbols` reduced skeleton tokens to about `48.41%`

### Practical interpretation

For code or project-directory inputs:

- `tree` is the lightest structural navigation view
- `imports` is useful when dependency shape matters more than implementation detail
- `symbols` is useful when exported surface and callable structure matter more than dependency wiring

In this fixture, the three focused directory views all landed at roughly the same compression level.
That means the operator can choose based on **reading intent**, not just token budget.

## Text Focus Results

The text-oriented result is very different.

`writing-outline` did **not** materially shrink the skeleton against `full`.

Typical ratios:

- char ratio: about `1.0030` to `1.0041`
- token ratio: about `0.9812` to `1.0044`

In plain language:

- sometimes `writing-outline` is very slightly smaller
- sometimes it is very slightly larger
- the difference is tiny either way

### Why this makes sense

For long-form text:

- `writing-outline` is not primarily a token minimization mode
- it is a **reading and editing mode**
- it makes the article/book structure easier to inspect without changing restore fidelity

So the product meaning is:

- `writing-outline` is mainly about better handoff structure
- `tree` / `imports` / `symbols` are where we currently see more meaningful skeleton-size reductions

## Main Conclusions

### 1. Directory focus modes are already valuable

On the quick fixture:

- directory focus modes reduced skeleton chars to about `48%` to `50%` of `full`
- directory focus modes reduced skeleton tokens to about `48%` to `50%` of `full`

That is a real reduction, not just a cosmetic output change.

### 2. Writing-outline is mostly a readability feature

For text cases:

- the token effect is close to neutral
- the real benefit is structural clarity for editorial workflows

### 3. Focus mode should be treated as a second-stage optimization

The current product stack now looks like this:

1. `context compress`
2. optional `--incremental`
3. optional `--focus-mode`

That means:

- full vs incremental controls **how much source surface** is transported
- focus mode controls **how that skeleton is presented**

For large repositories, the biggest transport win still comes from:

- incremental change surfaces

For operator experience, the next win comes from:

- picking the right focus mode for the task

## Recommended Usage

Use `tree` when:

- you want the fastest mental map of a project surface
- you care about paths and boundaries more than implementation

Use `imports` when:

- you want dependency wiring
- you are reasoning about module flow or integration edges

Use `symbols` when:

- you want exported functions, classes, and component structure
- you are handing context to implementation-oriented coding agents

Use `writing-outline` when:

- you are working on books, articles, or long drafts
- you want structural continuity more than raw compression gains

## Product Takeaway

This benchmark supports a careful claim:

- `focus-mode` is not just cosmetic
- for directory and code-oriented skeletons, it can materially reduce the AI-facing skeleton footprint
- for long-form writing, it is better understood as a structure-first viewing mode than as a hard compression mode

That is a healthy outcome.

It means `MCP-Skeleton` now has two distinct optimization layers:

1. **surface selection**
   - full vs incremental
2. **skeleton presentation**
   - full vs tree/imports/symbols/writing-outline
