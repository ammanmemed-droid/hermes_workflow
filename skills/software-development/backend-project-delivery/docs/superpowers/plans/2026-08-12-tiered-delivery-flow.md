# Tiered Backend Delivery Flow Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make mechanical backend changes follow a bounded fast path while requiring user approval before expanding into standard or high-risk work.

**Architecture:** Keep routing and the complete L1 contract in `SKILL.md`. Move detailed L2/L3 verification, security/data, service-discovery, AI, and project-path guidance into conditionally loaded references.

**Tech Stack:** Markdown Agent Skill, YAML interface metadata, Git.

## Global Constraints

- L1 uses precise search, minimal edit, old-value search, diff review, and at most one existing fast targeted check.
- Do not generate a full project tree or run full regression for L1.
- Stop and request one combined confirmation before expanding inspection or edits.
- Preserve unrelated user changes and generated-file boundaries.

---

### Task 1: Capture Baseline Behavior

**Files:**
- Create: `docs/superpowers/tests/2026-08-12-tiered-delivery-baseline.md`

- [ ] Run representative prefix and Nacos rename scenarios without the revised skill.
- [ ] Record whether the agent over-analyzes, plans, adds tests, runs broad regression, or expands without confirmation.
- [ ] Record the exact behavioral gaps the revision must correct.

### Task 2: Implement Tier Routing and L1 Contract

**Files:**
- Modify: `SKILL.md`

- [ ] Replace project-type-first routing with L1/L2/L3 task classification.
- [ ] Add bounded search, L1 operation budget, stopping conditions, and concise handoff format.
- [ ] Add a single confirmation gate for contract, consumer, production, security, data, or scope-expansion signals.
- [ ] Route L2/L3 and conditional references without loading them for L1.

### Task 3: Align Conditional References

**Files:**
- Modify: `references/delivery-paths.md`
- Modify: `references/verification-and-handoff.md`
- Modify: `references/ai-system-checks.md`
- Create: `references/security-and-data-changes.md`
- Create: `references/service-discovery-changes.md`

- [ ] State that project paths select engineering method, not task depth.
- [ ] Add minimum verification and final-report contracts by level.
- [ ] Add security/data and service-discovery guidance loaded only after L3 authorization.
- [ ] Ensure AI checks are loaded during design only when AI behavior, permissions, data, or side effects change.

### Task 4: Update Metadata and Remove Clutter

**Files:**
- Modify: `agents/openai.yaml`
- Delete: `README.md`

- [ ] Update UI metadata to describe risk-proportional execution and confirmation before escalation.
- [ ] Remove the unused placeholder README.

### Task 5: Validate Revised Skill

**Files:**
- Create: `docs/superpowers/tests/2026-08-12-tiered-delivery-results.md`

- [ ] Re-run representative L1 and escalation scenarios with the revised skill.
- [ ] Verify L1 stays bounded and escalation stops for confirmation.
- [ ] Check frontmatter, links, metadata, placeholders, diff cleanliness, and repository status.
- [ ] Record any limitations caused by higher-priority runtime instructions.
