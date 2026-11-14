# Plan: Pre-Commit Review Skill (`pre-commit-review`)

## 1. Executive Summary & Objectives

The goal is to design, structure, and document our first custom skill: **`pre-commit-review`**. 
This skill automates a rigorous, multi-angle subagent-driven review of staged changes *before* committing. 

Crucially, **this review applies to all staged changes**—not just code, but also documentation, specifications, configuration, and architectural artifacts, which is paramount in the AI-driven development era where specs and docs drive implementation accuracy.

### Addressing Agent Reliability (The "Lazy Agent" Problem)
◆︎ You are absolutely correct. A 40-second "deep investigation" resulting in "Zero findings" without any demonstration of the research process is 
not a rigorous review; it is an artifact of the agent taking the path of least resistance.

★ Insight ─────────────────────────────────────
- Agent Slacking (The "Lazy Agent" Pattern): When agents are given open-ended tasks like "perform deep investigation," they often default to a 
high-level summary if not forced to show their work.
- Enforcement Mechanisms: To ensure reliability, we must shift the agent prompt from "please investigate" to "you must document your 
investigation steps."
─────────────────────────────────────────────────

How we can fix this in the pre-commit-review skill:

To force the agents to actually work, we can modify the subagent prompt files (skills/pre-commit-review/prompts/) to include a mandatory "Research Log" section.

Here are the specific changes we should apply to the prompt files:

1. Mandatory Research Log: Require agents to fill out a block at the top of their report detailing:
* Files Searched: List the files or paths they examined.
* Context Queried: Briefly state what they looked for (e.g., "Compared openspec/config.yaml with openspec/specs/pre-commit-review/spec.md to 
ensure schema consistency").
* Hypotheses Tested: What potential "missed" things did they check for? (e.g., "Checked for missing gitignores for new directories").
2. Chain-of-Thought Requirement: Force the agent to write its reasoning before the conclusion.
3. Traceability: If the report doesn't contain a detailed log, the orchestrator (which you control) can flag the result as "incomplete" and 
force a re-run.

### Refined Governance Rules & Architectural Decisions:
1. **Bullet Points Over Tables**: To avoid markdown table parsing hazards, corruption, or pipe alignment errors, subagents will use **structured bullet-point blocks** below each finding rather than tables. V1 and V2 append their notes under dedicated headings for each finding, preserving raw text integrity.
2. **Review Directory Naming Convention (`docs/review/<datetime>_<slug>/`)**: All review runs MUST initialize review directories using the mandatory timestamped slug format `docs/review/<datetime>_<slug>/` (e.g., `docs/review/20261003_194935_pre_commit_review/`), ensuring chronological traceability and preventing collisions.
3. **Unstaged Files & Atomic Commits**: If no files are staged (`git diff --cached` is empty), the agents will analyze the unstaged changes, propose an atomic commit structure, stage the files automatically (with user confirmation/awareness), and proceed with the review protocol.
4. **Atomic Commit Principle & Huge Commits**: Huge, monolithic commits are discouraged as an anti-pattern. If a huge commit is detected, the orchestrator warns the user that commits should be atomic, and proceeds **only if the user explicitly confirms it is intentional**.
5. **Developer Responsibility / Freeze**: Reviewing active changes assumes the developer keeps the workspace stable/frozen during the review.
6. **Pre-Created Files via Bash**: The main orchestrator initializes the review directory and pre-creates report files by copying the shared template via Bash (`cp skills/pre-commit-review/templates/finding_report.md docs/review/<datetime>_<slug>/missed.md`, etc.). **Subagents must not create files themselves**.
7. **Dedicated Subagent System Prompt Files**: Each subagent (MISSED, MISINTERPRETED, LOST, V1, V2) has a dedicated system prompt file stored under `skills/pre-commit-review/prompts/`.
8. **Zero Fake Findings & Automatic Retry**: All 5 agents must perform thorough verification. If no issues are found, they state "Zero findings". If any subagent fails due to connection or runtime errors, the orchestrator **must re-run it automatically** until successful results are obtained.
9. **Mandatory Subagent Trace & Orchestrator Verification**: Subagents MUST explicitly write their findings (or explicit "Zero findings" statement) into their assigned report files (`docs/review/<datetime>_<slug>/missed.md`, etc.). The orchestrator MUST verify that the report file has been non-trivially modified and contains the agent's trace before proceeding. An empty, unwritten, or untouched template report is treated as a failed subagent execution, triggering an automatic re-run.
10. **Comprehensive Context Research**: Subagents are not restricted to a rigid list of files. They must proactively perform active research across the codebase, documentation, specs, and commit history to discover all related context, treating review as a deep investigation rather than a mechanical task.

---

## 2. Architectural Design & Pipeline

The review workflow executes strictly **sequentially** to prevent inference degradation, cross-agent contamination, and prompt clutter.

```
[ Pre-flight check: Check staged changes (if none, propose atomic staging from unstaged) & check commit size ]
                 │
                 ▼
 ┌──────────────────────────────────────────────┐
 │ PHASE 1: Primary Analysis Agents (Sequential)│
 │  1. MISSED Agent        (Reads missed_prompt.md)        │
 │  2. MISINTERPRETED Agent(Reads misinterpreted_prompt.md)│
 │  3. LOST Agent          (Reads lost_prompt.md)          │
 └──────────────────────┬───────────────────────┘
                        │
                        ▼
 ┌──────────────────────────────────────────────┐
 │ PHASE 2: Validation & Decision (Sequential)  │
 │  4. V1 Groundness Agent (Reads v1_prompt.md) │
 │     - Appends groundness comments under each  │
 │       finding block.                         │
 │  5. V2 Senior Decision Agent (Reads v2_prompt.md)│
 │     - Evaluates all 3 annotated reports.     │
 │     - Records Apply/Drop decisions.          │
 │     - Writes sorted final report             │
 │       to docs/review/<datetime>_<slug>/final_report.md │
 └──────────────────────┬───────────────────────┘
                        │
                        ▼
 ┌──────────────────────────────────────────────┐
 │ PHASE 3: Orchestrator Implementation         │
 │  - Main orchestrator applies approved items  │
 └──────────────────────┬───────────────────────┘
```

---

## 3. Directory Layout & Artifacts Structure

The skill and its templates/prompts will reside under `/skills/pre-commit-review/`:

```
skills/
└── pre-commit-review/
    ├── SKILL.md                 # Skill definition, orchestration instructions, and CLI invocation
    └── templates/
        ├── finding_report.md    # Single shared template using bullet points for MISSED, MISINTERPRETED, and LOST reports
        └── final_report.md      # Template for V2 Senior Decision Agent final report
    └── prompts/
        ├── missed_prompt.md     # System prompt for MISSED agent
        ├── misinterpreted_prompt.md # System prompt for MISINTERPRETED agent
        ├── lost_prompt.md       # System prompt for LOST agent
        ├── v1_prompt.md         # System prompt for V1 Groundness agent
        └── v2_prompt.md         # System prompt for V2 Senior Decision agent
```

---

## 4. Updated Templates Specification

### 4.1. Shared Finding Report Template (`finding_report.md`)
Using clear markdown bullet points per finding to prevent table corruption.

```markdown
# [Agent Name] Report
- **Timestamp:** {{timestamp}}
- **Target Slug:** {{slug}}

## Findings

### F1: [Short Title]
- **Severity:** (Critical / High / Medium / Low)
- **Target Type:** (Code / Documentation / Specification / Configuration)
- **File / Path:** 
- **Description:** 
- **Evidence / Comparison:** 
- **Suggested Resolution:** 

#### V1 Groundness Audit & Comments
*(Appended by V1 Agent)*
- **Status:** (Grounded / Hallucinated / Misread Docs)
- **Comments:** 

#### V2 Decision
*(Appended by V2 Senior Agent)*
- **Verdict:** (**Apply** / **Drop**)
- **Rationale:** 
```

*(Note: If an agent finds zero issues after thorough verification, it writes: "**Zero findings.** Thorough verification confirmed no issues.")*

### 4.2. Final Decision Report Template (`final_report.md`)
Organized into strict severity sections (`Critical`, `High`, `Medium`, `Low`) sorted by V2, stored in `docs/review/<datetime>_<slug>/final_report.md`.

```markdown
# Pre-Commit Final Review Decision Report (V2 Senior Agent)
- **Timestamp:** {{timestamp}}
- **Review Slug:** {{slug}}

## Executive Summary
Total findings reviewed: X | Approved to Apply: Y | Dropped: Z

---

## 1. Critical Severity Findings
### [ID] ([Source Agent])
- **Target Type & Path:** 
- **Summary:** 
- **Decision:** **Apply** / **Drop**
- **Action / Implementation Notes:** 

---

## 2. High Severity Findings
### [ID] ([Source Agent])
- **Target Type & Path:** 
- **Summary:** 
- **Decision:** **Apply** / **Drop**
- **Action / Implementation Notes:** 

---

## 3. Medium Severity Findings
### [ID] ([Source Agent])
- **Target Type & Path:** 
- **Summary:** 
- **Decision:** **Apply** / **Drop**
- **Action / Implementation Notes:** 

---

## 4. Low Severity Findings
### [ID] ([Source Agent])
- **Target Type & Path:** 
- **Summary:** 
- **Decision:** **Apply** / **Drop**
- **Action / Implementation Notes:** 

---

## 5. Dropped Findings Summary
| Finding ID | Source Agent | Severity | Reason for Dropping |
|---|---|---|---|
| | | | |
```

---

## 5. Step-by-Step Implementation Plan

1. **Scaffold Directory Structure**: Create `skills/pre-commit-review/templates/` and `skills/pre-commit-review/prompts/`.
2. **Write Templates**: Author `finding_report.md` and `final_report.md`.
3. **Write Subagent Prompt Files**: Author the 5 prompt files under `skills/pre-commit-review/prompts/`.
4. **Author Skill Definition (`SKILL.md`)**: Write the complete skill instruction file containing pre-flight checks (unstaged handling, atomic commit warnings), orchestrator Bash initialization (`cp`), prompt loading, subagent delegation with retry resilience, and implementation steps.
5. **Verification**: Verify all files are in place and correctly formatted.
