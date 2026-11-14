<!-- #region -->
# Specification Driven Development

> You have to run slow to run fast
>
> -- https://learn.deeplearning.ai/courses/spec-driven-development-with-coding-agents

# Workflow

1. Constitution
2. Feature phase N
   - Specification
   - implementation
   - validation
3. Replanning
4. Step 1.

# 1. Constitution

```bash
$ tree docs/specs/constitution/
constitution/
├── mission.md             # the core idea
├── roadmap.md             # features are planned
└── tech-stack.md

1 directory, 3 files
```

## Mission

- core idea
- target audience
- what success looks like

## Roadmap

- Phases
- What todo but not planned yet

High-level implementation order, in very small phases of work - 1-3 features in an iteration. Remember about the cognitive debt problem - all changes must be manageable for the human reviewer.

## Skill - VR

`consitution-writing` + validation (questions we have missed, inconsistencies)

# 2. Feature Specification

## Skills - VR

### Feature planning

`feature-planning`: reading constitution, taking the first undone feature, creating a spec + validation (consistency to the project's constituion, architectural docs, found problems, etc.)

"""
> IMPORTANT: Ask user questions BEFORE writing any files.

Find the next phase on docs/specs/roadmap.md and make a git worktree, ask me about the feature spec.

```bash
datetime=$(date +%Y%m%dT%H%M%S)
mkdir -p "docs/specs/features/feat_${datetime}_<slug>/review"
```

In there:

```bash
touch "docs/specs/features/feat_${datetime}_<slug>/{plan,requirements}.md"
```

Where:
  - `plan.md` is a series of numbered task groups.
  - `requirements.md` is the scope, decisions, context
  - `review` is the dir for keeping review files during the feature implementationn

Refer to `docs/specs/` files for guidance.  

Standards:
- TDD first approach - RED/GREEN/REFACTOR phases
- over-engineering adversary - cognitive load and maintenance perpsective!
"""

### Feature implementation

`feature-implementation`

### Feature validation

# Model-agnostic

- `AGENTS.md` for rules
- MCP for connecting to services
- Skills for workflow
- ACP (Agent Client Protocol) for connecting to different coding clients

# Open Questions - VR

1. gherkin - is it really needed for the project specification?
2. cognitive load - is a problem or we just need to let the models go by verifying the app behavior we required?
3. CLI tools for connecting to telegram and forgejo
<!-- #endregion -->
