# Contribution 1: `Extract settings access policy (OpenHands frontend refactor)`

**Contribution Number:** 1
**Student:** Sreytouch Lang (Jessica)
**Issue:** [OpenHands/OpenHands#15064](https://github.com/OpenHands/OpenHands/issues/15064)
**Status:** Phase I Complete

> **Claimability note (as of July 10, 2026):** Issue `#15064` is **open**, **unassigned**, and shows **no linked pull request** in the issue's Development section. The thread contains architecture discussion from the issue author, but no competing contributor claim comment, which makes it a clean Phase I target.

**Phase I checklist completed:** prepared and posted an introduction comment on the GitHub issue, verified the issue is live and claimable, confirmed the project is actively maintained with usable setup docs, verified my public fork exists under my GitHub account, and completed this Contribution README.

---

## Submission Evidence

- **Selected issue:** [OpenHands/OpenHands#15064 — "Architecture: extract settings access policy"](https://github.com/OpenHands/OpenHands/issues/15064)
- **Issue labels:** `settings`, `frontend`, `tech debt`, `type: refactor`, `Needs Design`
- **My issue comment (intro + interest):** [my intro comment on #15064 (jessicalang2595)](https://github.com/OpenHands/OpenHands/issues/15064#issuecomment-4942570892)
- **My fork:** [jessicalang2595/OpenHands](https://github.com/jessicalang2595/OpenHands)
- **Week 1 submission repo:** [Week1_PhaseI_Issue_Selection_SreytouchLang](https://github.com/jessicalang2595/Week1_PhaseI_Issue_Selection_SreytouchLang)
- **Course issue sheet update:** completed in the AI301 course tracker

---

## Why I Chose This Issue

I chose this issue because it is a strong match for the kind of frontend work I do best: React, TypeScript, route logic, and refactoring large UI control flows into smaller testable modules. The core task is not inventing a brand-new feature, but restructuring an existing decision path into a pure policy module, which fits the kind of architecture and cleanup work I enjoy.

**Skill match:** this issue lives in frontend route access logic and redirect behavior, which means working with React Router, TypeScript control flow, and policy-style decision logic. Those are areas where I am comfortable reading existing code, tracing edge cases, and making behavior-preserving refactors.

**Learning goal:** I want to get better at extracting business rules from framework-heavy code into pure, unit-testable functions. That is a different skill from just adding UI features, and it is valuable because it makes future bug fixing and maintenance easier.

**Understanding:** I understand why this issue matters. If settings access rules are scattered through a procedural route loader, it becomes harder to reason about redirect behavior, harder to test role-based access decisions, and easier to break settings navigation when flags, organizations, or billing visibility rules change.

---

## Understanding the Issue

### Problem Summary (2–4 sentences)

The OpenHands settings route currently mixes framework wiring with business-policy decisions in one loader. It decides who can access which settings page, when a user should be redirected, and how feature flags, organization state, ACP mode, and billing visibility affect navigation, all inside procedural route logic. That matters because redirect and permission rules are core product behavior, and when those rules are coupled to query-cache reads and route-loader code, they become much harder to test cleanly. I chose this issue because it is a well-scoped frontend refactor with clear user impact and strong testability value.

### Expected Behavior (after the refactor)

- Settings access rules should live in a pure policy function such as `decideSettingsAccess(facts)`.
- The route edge should gather facts like config, pathname, selected organization, user role, and settings data, then pass them into the policy.
- The policy should return a clear result like render vs redirect, rather than performing routing decisions inline.
- The redirect matrix should be unit-testable without needing the full route loader, query cache, or store setup.

### Current Behavior

1. `frontend/src/routes/settings.tsx` owns route-level access and redirect logic directly inside the loader.
2. The loader mixes SaaS-only path guards, feature-flag hiding, ACP gating, selected-organization lookup, billing visibility checks, admin permission logic, and redirect selection.
3. Some helper functions already exist, but the orchestration and final decision-making still live in one procedural flow.
4. The loader also reads query-cache and organization state directly, which couples business rules to framework and data-loading details.

### Affected Components (specific files/modules likely involved)

- `frontend/src/routes/settings.tsx`
- `frontend/src/utils/settings-utils.ts`
- `frontend/src/hooks/use-settings-nav-items.ts`
- `frontend/src/utils/org/permission-checks.ts`
- `frontend/src/utils/org/billing-visibility.ts`

> **`Needs Design` note:** because this issue is labeled `Needs Design`, I plan to confirm the shape of the extracted policy with maintainers before implementation. My default direction is a pure decision function that receives plain facts and returns a render-or-redirect result, while keeping query-cache reads and route params at the loader edge.

---

## Reproduction Process

### Environment Setup

Per Phase I guidance, I focused on understanding and scoping the issue instead of claiming implementation work. I reviewed the issue itself, the OpenHands contributing guide, and the current frontend route files on `main`.

OpenHands' documented setup indicates the project uses:

- Docker
- Python 3.12
- Node.js 22+
- Poetry 1.8+
- the repository `README.md` / `CONTRIBUTING.md` workflow for local setup and test execution

### Steps to Confirm the Refactor Target

1. Open `frontend/src/routes/settings.tsx` on current `main`.
2. Trace the `clientLoader` logic and identify which access decisions it owns.
3. Confirm whether redirect rules are expressed inline or already isolated into a pure policy function.
4. Review related helper files to see what is already extracted and what still remains coupled to the route loader.

### Findings

The issue is real and current. The settings route already has some helper functions, but the final orchestration and redirect logic are still procedural and still tied to query-cache and selected-organization reads. That makes the issue a good candidate for a behavior-preserving extraction rather than a speculative rewrite.

---

## Solution Approach

### Analysis

This is not mainly a UI bug; it is a structure and testability problem. The root issue is that access policy and redirect decisions are mixed with loader-specific framework concerns, which makes the logic harder to test and maintain than it needs to be.

### Proposed Solution

Extract the settings access and redirect logic into a pure policy module that accepts plain facts and returns a decision object. Keep framework-specific work like `queryClient.getQueryData(...)`, route params, and selected-organization store reads in a thin loader boundary.

### Implementation Plan (UMPIRE, adapted)

**Understand:** the route should gather facts; the policy should decide behavior.

**Match:** reuse the existing settings helpers instead of replacing them wholesale, and refactor only the coupled orchestration layer.

**Plan:**
1. Confirm the desired policy shape with maintainers because the issue is labeled `Needs Design`.
2. Identify the minimal fact object needed for access decisions.
3. Extract a pure `decideSettingsAccess(...)` function.
4. Update the route loader so it gathers facts and delegates to the policy.
5. Add focused unit tests that cover redirect and permission cases without React Router loader setup.

**Implement:** branch and commit details will be added in Phase II / III.

**Review:**
- Does the extracted policy preserve existing redirect behavior?
- Is the function framework-free and easy to test in isolation?
- Did the route loader get simpler rather than just moving complexity around?
- Are admin-only, billing, hidden section, and ACP scenarios still covered?

**Evaluate:** run focused tests for the policy and confirm the route behavior remains unchanged.

---

## Testing Strategy

### Unit Tests

- [ ] `decideSettingsAccess(...)` returns render or redirect correctly for normal settings routes
- [ ] SaaS-only paths redirect correctly
- [ ] hidden settings sections redirect correctly based on feature flags
- [ ] ACP gating behavior is preserved
- [ ] billing visibility and admin-only page behavior are preserved

### Integration Tests

- [ ] Route loader still behaves correctly after delegating to the policy
- [ ] Existing settings navigation behavior remains unchanged from the user perspective

### Manual Testing

Navigate through settings routes with different organization or permission states and confirm the UI lands on the expected page.

---

## Acceptance Criteria — what "fixed" looks like (concrete)

1. A pure settings access policy module exists and is used by the settings route.
2. Redirect decisions are no longer expressed inline inside the main route loader flow.
3. Query-cache and store reads remain at the route edge rather than inside the policy.
4. Unit tests cover the redirect matrix without needing full route-loader setup.
5. User-visible settings behavior stays the same after the refactor.

---

## Implementation Notes

### Week 1 Progress

- Verified that `#15064` is open
- Verified that no one is currently assigned
- Verified that the issue shows no linked pull request
- Reviewed the relevant frontend settings-route files
- Confirmed the project is active and has usable setup docs
- Prepared my Phase I issue claim and completed this README

### Selection Quality Assessment

This issue is cleaner than my earlier contested target because it is still open, unassigned, and not already tied to a PR. It is also specific enough to understand in Phase I without being so broad that the work becomes vague or unbounded.

### Code Changes

- **Files modified:** README only during Phase I
- **Key commits:** to be added in later phases
- **Approach decisions:** keep the scope narrow, preserve behavior, and optimize for a pure testable policy extraction

---

## Pull Request

**PR Link:** To be added in Phase IV.

**PR Description (planned):** Extract settings route access and redirect decisions into a pure settings access policy, keep fact gathering at the route edge, and add focused unit tests for the redirect matrix.

**Maintainer Feedback:** none yet — I will record any design clarification here if maintainers respond.

**Status:** Phase I complete, ready for Phase II investigation and design confirmation.

---

## Learnings & Reflections

### Technical Skills Gained

This issue already pushed me to think more carefully about the difference between helper extraction and true policy extraction. I also got more practice reading route-loader logic and identifying where framework concerns should stop and business rules should begin.

### Challenges Overcome

The biggest challenge was making sure I picked a truly claimable issue rather than one that looked good at first glance but had already become contested or assigned. I also had to separate what the issue says at a high level from what the current code actually does.

### What I'd Do Differently Next Time

I would keep verifying claimability as close to submission time as possible, not just at initial selection time, because issue state can change quickly.

---

## Resources Used

- [CodePath AI301 Phase I instructions](https://courses.codepath.org/courses/ai301/unit/1#!projects)
- [OpenHands issue #15064](https://github.com/OpenHands/OpenHands/issues/15064)
- [OpenHands contributing guide](https://github.com/OpenHands/OpenHands/blob/main/CONTRIBUTING.md)
- [OpenHands settings route file](https://github.com/OpenHands/OpenHands/blob/main/frontend/src/routes/settings.tsx)
