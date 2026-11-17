# Конспект курса SDD — как записан

DeepLearning.AI, «Spec-Driven Development with Coding Agents»:
<https://learn.deeplearning.ai/courses/spec-driven-development-with-coding-agents>.
По-английски, как записан, без правок; поправки — в отметке ниже и в остальных
файлах каталога.

> **Отметка 2026-11-16 — что изменилось после записи.** В разделе «Feature
> planning» ниже сказано, что `requirements.md` — это «the scope, decisions,
> context», и что каталог задачи содержит `{plan,requirements}.md`. Решено иначе:
> требования ставит постановщик задачи, а проработка — гипотезы, решения с
> отклонёнными вариантами, границы — каталог `research/` в каталоге задачи; этот
> файл — его `README.md`. В разделе «1. Constitution» ниже сказано, что конституция
> «is read on every task»: в каждую сессию грузится только `AGENTS.md`, остальное —
> по необходимости.

## Workflow

1. Constitution
2. Feature phase N
   - Specification
   - implementation
   - validation
3. Replanning
4. Step 1.

## 1. Constitution

```bash
$ tree docs/constitution/
docs/constitution/
├── mission.md             # the core idea
├── roadmap.md             # features are planned
└── tech-stack.md

1 directory, 3 files
```

The constitution lives at `docs/constitution/`, not under `docs/specs/`:
`docs/specs/` holds the materials of one work item, while the constitution is
always current and is read on every task.

### Mission

- core idea
- target audience
- what success looks like

### Roadmap

- Phases
- What todo but not planned yet

High-level implementation order, in very small phases of work - 1-3 features in an iteration. Remember about the cognitive debt problem - all changes must be manageable for the human reviewer.

### Skill - VR

`constitution-writing` + validation (questions we have missed, inconsistencies)

## 2. Feature Specification

### Skills - VR

#### Feature planning

`feature-planning`: reading constitution, taking the first undone feature, creating a spec + validation (consistency to the project's constituion, architectural docs, found problems, etc.)

"""
> IMPORTANT: Ask user questions BEFORE writing any files.

Find the next phase on docs/constitution/roadmap.md and make a git worktree, ask me about the feature spec.

```bash
datetime=$(date +%Y%m%dT%H%M%S)
mkdir -p "docs/specs/features/feat_${datetime}_<slug>"
```

In there:

```bash
touch "docs/specs/features/feat_${datetime}_<slug>/{plan,requirements}.md"
```

Where:
  - `plan.md` is a series of numbered task groups.
  - `requirements.md` is the scope, decisions, context

Refer to `docs/specs/` files for guidance.  

Standards:
- TDD first approach - RED/GREEN/REFACTOR phases
- over-engineering adversary - cognitive load and maintenance perpsective!
"""

#### Feature implementation

`feature-implementation`

#### Feature validation

## Conventions

- Documents are written in Russian. Code, paths, identifiers, JSON keys and
  commit-message verbs stay untranslated.
- A constitution holds only what is unique to its repository. Team-wide
  practices (SDD, TDD, model-agnostic, over-engineering) live in skills and
  apply everywhere; a constitution does not repeat them.

Nothing else here yet. The lifecycle of a spec - where its plan lives, whether it
is one file or a directory, archival, review location - is open question 4 and
must not be encoded before it is settled.

## Model-agnostic

- `AGENTS.md` for rules
- MCP for connecting to services
- Skills for workflow
- ACP (Agent Client Protocol) for connecting to different coding clients

## Open Questions - VR

1. gherkin - is it really needed for the project specification?
2. cognitive load - is a problem or we just need to let the models go by verifying the app behavior we required?
3. CLI tools for connecting to telegram
4. specs lifecycle - one file or a directory, where the plan of a task lives,
   archival, review location. Not now: first the basic repository documents,
   then finally the `qwen_code_cli` module. Small increments, iteratively.
   Evidence to weigh: `misc/plan_20261003_194935_pre_commit_review.md`
   is already tracked - the single-file convention has left a file in the repo.

