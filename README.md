# Contribution 1: `Extract conversation WebSocket event interpreter (OpenHands frontend refactor)`

**Contribution Number:** 1
**Student:** Sreytouch Lang (Jessica)
**Issue:** [OpenHands/OpenHands#15061](https://github.com/OpenHands/OpenHands/issues/15061)
**Status:** Phase I Complete

> **Claimability note (as of July 6, 2026):** Issue `#15061` is **open**, **unassigned**, has **no linked pull request**, and no other contributor has claimed it. It is on `OpenHands/OpenHands`, an actively maintained project with a public contributing guide and working local-setup docs. This is a clean, uncontested Phase I target — I verified all four claimability conditions (open / unassigned / no linked PR / no claim comment) before selecting it.

**Phase I checklist completed:** commented on the GitHub issue introducing myself and expressing interest, updated the course issue sheet, forked `OpenHands/OpenHands`, and completed this Contribution README.

---

## Submission Evidence

- **Selected issue:** [OpenHands/OpenHands#15061 — "Architecture: extract conversation WebSocket event interpreter"](https://github.com/OpenHands/OpenHands/issues/15061)
- **Issue labels:** `frontend`, `type: refactor`, `tech debt`, `Needs Design`
- **My issue comment (intro + interest):** _paste your comment permalink here after posting — format `https://github.com/OpenHands/OpenHands/issues/15061#issuecomment-XXXXXXXX`_
- **My fork:** [my OpenHands fork](https://github.com/jessicalang2595/OpenHands)
- **Week 1 submission repo:** [Week1_PhaseI_Issue_Selection_SreytouchLang](https://github.com/jessicalang2595/Week1_PhaseI_Issue_Selection_SreytouchLang)
- **Course issue sheet update:** completed in the AI301 course tracker

---

## Why I Chose This Issue

I chose this issue because it is a direct, honest extension of skills I have already demonstrated. In prior work I traced the OpenHands V1 conversation WebSocket path end to end — `conversation-websocket-context.tsx`, `ws-client-provider.tsx`, `use-send-message.ts`, and the handler test suite — so I already understand how this module receives raw socket events, interprets them, and updates conversation state. That prior investment means I can be productive quickly instead of spending the whole contribution just orienting myself.

**Skill match:** the work is squarely in my strongest area — React, TypeScript, and WebSocket lifecycle logic. Extracting an event interpreter is a pure separation-of-concerns refactor: pull the "how do I interpret this incoming event" logic out of the context component and into an isolated, unit-testable module.

**Learning goal:** I want to get better at *refactoring for testability* on a real, large codebase — turning tangled component logic into a pure, independently testable unit without changing behavior. That is a distinct and valuable skill from writing new features, and it is exactly what this issue teaches.

**Understanding:** I understand *why* the maintainers filed this. The conversation WebSocket context currently does two jobs at once — it manages the socket connection lifecycle **and** it interprets/routes every incoming event. That coupling makes the interpretation logic hard to test in isolation and makes the context harder to reason about. Extracting the interpreter reduces that coupling and unlocks focused unit tests, which is why it is labeled `tech debt` + `type: refactor`.

---

## Understanding the Issue

### Problem Summary (2–4 sentences)

The OpenHands frontend conversation WebSocket context both manages the socket connection lifecycle *and* interprets incoming WebSocket events inline, so event-interpretation logic is coupled to a large React context and cannot be unit-tested on its own. This matters because interpretation is core business logic — how each server event is parsed and routed determines what the user sees — yet today it can only be exercised indirectly through the full context, making regressions easy to introduce and hard to catch. The issue asks to **extract the event-interpretation logic into a separate, pure module** so the context only wires things together and the interpreter can be tested directly. I chose it because it is well-scoped, uncontested, and lands on code I already understand.

### Expected Behavior (after the refactor)

- Incoming WebSocket event interpretation lives in a dedicated, pure module (e.g. a `conversation-websocket-event-interpreter`) with a clear input → output contract.
- The conversation WebSocket context imports and delegates to that interpreter instead of inlining the logic.
- **No user-visible behavior changes** — this is a structural refactor. Events are interpreted and routed exactly as before.
- The extracted interpreter has its own focused unit tests.

### Current Behavior

1. `conversation-websocket-context.tsx` contains both connection-lifecycle management and inline event interpretation/routing.
2. The interpretation logic is only reachable through the full context, so tests must stand up the whole context to exercise it.
3. This coupling is flagged by maintainers as `tech debt` and a `type: refactor` opportunity.

### Affected Components (specific files/modules likely involved)

- `frontend/src/contexts/conversation-websocket-context.tsx` — source of the extraction (interpretation logic to move out)
- `frontend/src/contexts/` — likely home for a new `conversation-websocket-event-interpreter.ts` (name TBD with maintainers, given the `Needs Design` label)
- `frontend/__tests__/conversation-websocket-handler.test.tsx` — existing coverage that must keep passing
- A new interpreter unit-test file for the extracted module
- Any event/type definitions the interpreter depends on (imported, not duplicated)

> **`Needs Design` note:** this label means the maintainers want to agree on the *shape* of the extracted module before implementation. My Phase I plan accounts for this: I will propose an interpreter interface in the issue and confirm it with a maintainer before writing the refactor, rather than guessing.

---

## Reproduction Process

### Environment Setup

Per Phase I guidance, I am focusing on issue selection, understanding, and planning rather than claiming completed implementation. I reviewed the issue, the OpenHands contributing guide, and the relevant frontend files on current `main`. Because this is a refactor (not a runtime bug), "reproduction" here means confirming the coupling exists in the current source, which I did by reading `conversation-websocket-context.tsx`.

OpenHands' contributing guide indicates local setup requires:

- Linux, macOS, or WSL
- Docker
- Python 3.12
- Node.js 22+ (repo requires `>=22.12.0`)
- Poetry 1.8+
- `make build` / `make run`; frontend tests via the repo's Vitest setup

### Steps to Confirm the Refactor Target

1. Open `frontend/src/contexts/conversation-websocket-context.tsx` on current `main`.
2. Locate where incoming WebSocket messages/events are parsed and routed to handlers/state updates.
3. Confirm that this interpretation logic is defined inline in the context rather than in a separate, independently importable module.
4. Confirm the handler tests exercise this logic only through the full context.

### Findings

The interpretation logic is coupled to the context, which is exactly the `tech debt` the issue targets. The clean seam is the point where a raw socket event enters and is turned into an application action/state update — that is what should become the pure interpreter.

---

## Solution Approach

### Analysis

This is a coupling/testability problem, not a functional bug. The root cause: interpretation logic lives inside a React context alongside unrelated connection-lifecycle concerns. The fix is to identify the pure "event in → action out" boundary and lift it into its own module with an explicit contract, leaving the context as a thin coordinator.

### Proposed Solution

Extract the conversation WebSocket event-interpretation logic into a dedicated, pure, dependency-light module and have the context delegate to it. Preserve behavior exactly; add focused unit tests for the new interpreter; keep the existing handler tests green.

### Implementation Plan (UMPIRE, adapted)

**Understand:** The context should keep *only* connection lifecycle + wiring; interpreting an incoming event into an app action/state change is a pure function that belongs in its own module.

**Match:** Reference the existing structure and test patterns already in the repo (`conversation-websocket-context.tsx`, `conversation-websocket-handler.test.tsx`) so the extraction matches house style rather than inventing a new pattern.

**Plan:**
1. Propose the interpreter interface in the issue (respecting the `Needs Design` label) and get maintainer sign-off on shape/naming.
2. Create the new interpreter module with an explicit input (raw event) → output (interpreted action/state update) contract.
3. Move interpretation logic out of the context into the module, no behavior change.
4. Update the context to import and delegate to the interpreter.
5. Add unit tests for the interpreter; run the existing handler suite to prove no regression.

**Implement:** Branch + commit links added in Phase II/III once the design is confirmed.

**Review:**
- Is behavior identical (no user-visible change)?
- Is the interpreter genuinely pure and testable in isolation?
- Do existing handler tests still pass unchanged?
- Is the context now meaningfully thinner/clearer?

**Evaluate:** Run the targeted Vitest suite plus the new interpreter tests under the repo's required Node version, and confirm no behavioral diff.

---

## Testing Strategy

### Unit Tests
- [ ] Interpreter returns the correct interpreted action for each incoming event type.
- [ ] Interpreter handles malformed/unknown events safely (matches current behavior).
- [ ] Interpreter is exercised directly, without standing up the full context.

### Integration Tests
- [ ] Existing `conversation-websocket-handler.test.tsx` suite passes unchanged after the context delegates to the interpreter.
- [ ] End-to-end event flow through the context produces identical state updates to pre-refactor `main`.

### Manual Testing
Open a conversation, exercise the normal event flow, and confirm the UI behaves identically to `main`.

---

## Acceptance Criteria — what "fixed" looks like (concrete)

1. A new, pure event-interpreter module exists and is imported by `conversation-websocket-context.tsx`.
2. Interpretation logic no longer lives inline in the context.
3. The interpreter has its own passing unit tests that do **not** require the full context.
4. The existing handler test suite passes **unchanged**.
5. **Zero user-visible behavior change** — verified by diffing event-driven state updates against `main`.
6. Module shape/naming confirmed with a maintainer first (satisfies the `Needs Design` label).

---

## Implementation Notes

### Week 1 Progress
- Verified `#15061` is open, unassigned, has no linked PR, and no competing claim comment (clean, uncontested target).
- Read `conversation-websocket-context.tsx` on current `main` and confirmed the interpretation/lifecycle coupling the issue describes.
- Posted a public interest comment introducing myself on the issue.
- Forked `OpenHands/OpenHands` and updated the course issue sheet.

### Selection Quality Assessment
Unlike a contested issue, this one has **no existing PR and no assignee**, so it satisfies the "live, claimable issue" bar cleanly. The only nuance is the `Needs Design` label; I address it head-on by proposing the interpreter interface and confirming it with a maintainer before implementing, which turns the design step into part of the plan rather than a risk.

### Code Changes
- **Files modified:** README only during Phase I.
- **Key commits:** added after design sign-off in Phase II/III.
- **Approach decisions:** scope deliberately narrow — one clean extraction, behavior-preserving, test-backed.

---

## Pull Request

**PR Link:** To be added in Phase IV.
**PR Description (planned):** Extract the conversation WebSocket event-interpretation logic from `conversation-websocket-context.tsx` into a dedicated pure module with its own unit tests, preserving behavior and keeping the existing handler suite green.
**Maintainer Feedback:** none yet — will record the design-confirmation exchange here.
**Status:** Phase I complete, awaiting maintainer design sign-off before implementation.

---

## Learnings & Reflections

### Technical Skills Gained
Deeper practice reading a large React/TypeScript codebase to find a clean refactor seam, and reasoning about how to make coupled logic independently testable without changing behavior.

### Challenges Overcome
The main Phase I challenge was choosing a target that is both a strong skill match **and** genuinely claimable. I explicitly checked open/unassigned/no-linked-PR/no-claim-comment before committing, and I planned around the `Needs Design` label instead of ignoring it.

### What I'd Do Differently Next Time
Nothing changed about my process — I now verify all four claimability conditions up front, which is exactly what I did here.

---

## Resources Used
- [CodePath AI301 Phase I instructions](https://courses.codepath.org/courses/ai301/unit/1#!projects)
- [OpenHands issue #15061](https://github.com/OpenHands/OpenHands/issues/15061)
- [OpenHands contributing guide](https://github.com/OpenHands/OpenHands/blob/main/CONTRIBUTING.md)
- [conversation-websocket-context.tsx](https://github.com/OpenHands/OpenHands/blob/main/frontend/src/contexts/conversation-websocket-context.tsx)
- [conversation-websocket-handler.test.tsx](https://github.com/OpenHands/OpenHands/blob/main/frontend/__tests__/conversation-websocket-handler.test.tsx)
