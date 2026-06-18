# Contribution 1: `Add message queue support for V1 conversations during WebSocket connection`

**Contribution Number:** 1  
**Student:** Sreytouch Lang(Jessica) 
**Issue:** [OpenHands/OpenHands#12279](https://github.com/OpenHands/OpenHands/issues/12279)  
**Status:** Phase I Complete

**Important note as of June 7, 2026:** I already commented on this issue from `jessicalang2595` on June 3, 2026 Pacific time / June 4, 2026 UTC, and the issue is still open and labeled `good first issue`, `UI/UX`, and `frontend`. However, there is also an active related PR: [OpenHands/OpenHands#14692](https://github.com/OpenHands/OpenHands/pull/14692). That made this a riskier Phase I selection than a completely uncontested issue, so I want that tradeoff documented clearly.

**Phase I checklist completed:** commented on the GitHub issue, updated the course issue sheet, forked `OpenHands/OpenHands`, and completed the Phase I README requirements.

## Submission Evidence

- **Selected issue:** [OpenHands/OpenHands#12279](https://github.com/OpenHands/OpenHands/issues/12279)
- **My issue comment:** [jessicalang2595 comment on #12279](https://github.com/OpenHands/OpenHands/issues/12279#issuecomment-4618034397)
- **My fork:** [jessicalang2595/OpenHands](https://github.com/jessicalang2595/OpenHands)
- **Week 1 submission repo:** [jessicalang2595/Week1_PhaseI_Issue_Selection_SreytouchLang](https://github.com/jessicalang2595/Week1_PhaseI_Issue_Selection_SreytouchLang)
- **Course issue sheet update:** completed in the private course tracker used for AI301 submissions

---

## Why I Chose This Issue

I chose this issue because it matches my frontend and full-stack interests very well. It sits at the intersection of React, TypeScript, WebSocket behavior, asynchronous state handling, and user experience. The bug is concrete and easy to explain: users should be able to send a message while the runtime is still starting up, and the app should safely deliver that message once the connection is ready.

This issue also feels valuable from a product perspective, not just a code perspective. Messaging reliability is part of user trust. If someone types a message during startup and it disappears, the app feels broken even if the backend eventually recovers. Working on this would help me practice debugging connection lifecycle behavior while improving a real user-facing workflow in OpenHands.

---

## Understanding the Issue

### Problem Description

V1 conversations in OpenHands do not reliably handle messages sent before the native WebSocket connection is fully established. During sandbox or runtime startup, a user may attempt to send a message before the socket is ready. That message should be queued and delivered automatically once the connection opens, but the current V1 flow does not consistently provide that experience.

### Expected Behavior

When a user submits a message before the WebSocket is connected, the app should preserve that message, queue it in order, and automatically deliver it once the V1 conversation socket becomes ready. This should match the behavior already supported in the V0 flow.

### Current Behavior

Based on the issue and the current linked PR discussion:

1. V1 conversations do not provide the expected queue-and-flush experience during early connection startup.
2. Messages created during socket startup or reconnect can be delayed, blocked, or not immediately delivered.
3. The V0 implementation already has a pending-event pattern, but the V1 implementation still needs equivalent or improved handling.

### Affected Components

- `frontend/src/contexts/conversation-websocket-context.tsx`
- `frontend/src/context/ws-client-provider.tsx`
- `frontend/src/hooks/use-send-message.ts`
- `frontend/__tests__/conversation-websocket-handler.test.tsx`
- WebSocket `onOpen` lifecycle behavior
- pending-message or pending-event queue logic for V1 conversations

---

## Reproduction Process

### Environment Setup

Per CodePath Phase I guidance, I am focusing first on issue selection, issue understanding, and contribution planning rather than claiming that I have already completed local implementation work. I reviewed the public issue details, the OpenHands contribution guide, the relevant frontend file references listed in the issue, and the currently open PR connected to this bug.

OpenHands' contributing guide indicates the local setup will require:

- Linux, macOS, or WSL
- Docker
- Python 3.12
- Node.js 22+
- Poetry 1.8+
- `make build`
- `make run`

### Steps to Reproduce

1. Open a V1 conversation in OpenHands while the sandbox or runtime is still starting.
2. Attempt to send a user message before the native WebSocket is fully connected.
3. Observe whether the message is queued and automatically delivered when the socket opens.
4. Compare that behavior against the expected V0-style queue-and-flush experience.

### Reproduction Evidence

- **Commit showing reproduction:** Not required in Phase I. This would be added in Phase II after local reproduction work begins.
- **Screenshots/logs:** Not required in Phase I. These would be added in Phase II if the UI or WebSocket logs clearly show the timing problem.
- **My findings:** The issue body points to V1 conversation WebSocket handling as the main gap, and the open PR suggests one concrete fix path: explicitly flushing pending messages in the WebSocket `onOpen` handler.

---

## Solution Approach

### Analysis

This appears to be a connection-lifecycle bug in the V1 frontend conversation flow. The main requirement is not just storing pending messages, but ensuring they are flushed at the correct moment when the WebSocket becomes usable. The issue body points to the V0 `pendingEventsRef` and `flushPendingEvents` approach as a helpful reference, while the currently open PR proposes explicitly calling a pending-message delivery function inside the V1 WebSocket `onOpen` path.

That means the likely root cause is one of these:

- V1 never queues messages correctly before connection
- V1 queues them but never explicitly flushes them when the socket opens
- V1 relies on a slower or indirect delivery path instead of an immediate client-side flush

### Proposed Solution

Implement V1 message queuing and guaranteed flush-on-open behavior so pending user messages are preserved and delivered in order once the WebSocket becomes available. The implementation should include test coverage for startup timing, reconnect behavior, and queue clearing when the conversation changes or stops.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:**  
V1 conversations should behave like V0 for early message submission: if a user sends a message before the WebSocket is ready, that message should be queued and then automatically delivered when the connection opens.

**Match:**  
The issue already points to a likely reference implementation in V0:

- `frontend/src/context/ws-client-provider.tsx`
- `pendingEventsRef`
- `flushPendingEvents`

The V1 implementation appears to live in:

- `frontend/src/contexts/conversation-websocket-context.tsx`
- `frontend/src/hooks/use-send-message.ts`

**Plan:**
1. Review the V1 `sendMessage` and WebSocket connection lifecycle in `conversation-websocket-context.tsx`.
2. Compare V1 behavior with the V0 queue-and-flush implementation in `ws-client-provider.tsx`.
3. Identify whether V1 is missing queue creation, queue flushing, or both.
4. Add or adjust flush logic so queued messages are delivered immediately in the WebSocket `onOpen` handler.
5. Add tests in `frontend/__tests__/conversation-websocket-handler.test.tsx` for queueing, ordered delivery, reconnect behavior, and queue clearing.

**Implement:**  
Branch and commit links would be added in Phase II only if this issue remained a valid target after maintainer confirmation.

**Review:**  
- Does the fix preserve message order?
- Does it avoid duplicate delivery on reconnect?
- Does it clear stale queued messages when a conversation changes?
- Does test coverage include startup and reconnect edge cases?

**Evaluate:**  
I will verify the fix by reproducing startup timing locally, running the relevant frontend tests, and checking that queued messages are delivered as soon as the V1 WebSocket opens.

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: a message sent before V1 socket connection is queued instead of lost
- [ ] Test case 2: queued messages are flushed in order once the socket opens
- [ ] Test case 3: pending messages are cleared correctly when the conversation changes or stops

### Integration Tests

- [ ] Integration scenario 1: user sends a message during sandbox startup and it appears once the connection becomes ready
- [ ] Integration scenario 2: socket reconnect path flushes pending messages without duplicates

### Manual Testing

Manual testing in Phase II should include opening a V1 conversation, sending a message during startup, verifying successful delivery after socket connection, then repeating the same flow during a reconnect scenario.

---

## Implementation Notes

### Week 1 Progress

During Week 1, I reviewed the OpenHands issue, the project contribution guide, and the issue-linked files to understand the scope. I also verified that the issue is still open as of June 7, 2026 and that it is labeled `good first issue`, `UI/UX`, and `frontend`.

I also confirmed two important project-management details:

- I already left a public interest comment on the issue from my GitHub account.
- There is now an active open PR linked to the same issue: [OpenHands/OpenHands#14692](https://github.com/OpenHands/OpenHands/pull/14692), created on June 5, 2026 Pacific time / June 6, 2026 UTC.

Because of that open PR, this issue is useful for learning and README planning, but it is risky as a final contribution target unless a maintainer explicitly confirms that another implementation or follow-up is still needed.

I completed the Phase I contributor tasks by posting on the issue, tracking the selection in the course sheet, preparing my fork, and finishing this README with the issue summary and contribution rationale.

### Selection Risk Assessment

The strongest weakness in this Phase I submission is not missing work, but issue-selection risk:

1. the issue already had visible contributor activity
2. there was already an active related PR during my Week 1 planning
3. that made it a weaker Phase I target than a cleaner unclaimed issue

I am keeping that risk documented here because it is an important part of honestly evaluating the quality of my issue choice.

### Week 2 Progress

Planned for Phase II after maintainer confirmation or after selecting a backup issue in OpenHands.

### Code Changes

- **Files modified:** README only in this repository during Phase I
- **Key commits:** To be added after implementation work begins
- **Approach decisions:** I chose to document the real OpenHands issue I commented on, but I also recorded the active-PR risk clearly so I do not pretend the issue is uncontested.

---

## Pull Request

**PR Link:** To be added in Phase IV if I continue with this issue or switch to a closely related OpenHands issue.

**PR Description:**  
Planned draft: improve V1 conversation reliability by queueing messages sent before the WebSocket is ready and flushing them automatically once the connection opens, with test coverage for startup and reconnect behavior.

**Maintainer Feedback:**
- No direct maintainer feedback to my comment yet.
- Existing related PR from another contributor: [OpenHands/OpenHands#14692](https://github.com/OpenHands/OpenHands/pull/14692)
- My public interest comment is here: [issue comment link](https://github.com/OpenHands/OpenHands/issues/12279#issuecomment-4618034397)

**Status:** Phase I complete, awaiting next implementation step

---

## Learnings & Reflections

### Technical Skills Gained

So far, I have gotten more practice evaluating frontend async issues by reading issue bodies, tracing likely file ownership, and understanding how WebSocket connection timing can affect user-facing behavior.

### Challenges Overcome

The hardest part here was separating the actual issue from the linked implementation work. The OpenHands link you shared is a pull request, not the issue itself, so I had to trace it back to `#12279`. I also had to verify whether the issue was still open and whether someone else was already actively implementing it.

### What I'd Do Differently Next Time

Next time I would check three things immediately before committing to an issue:

1. whether it is an issue or a PR
2. whether there are recent contributor claim comments
3. whether there is already an open PR linked to the fix

---

## Resources Used

- [CodePath AI301 Phase I instructions](https://courses.codepath.org/courses/ai301/unit/1#!projects)
- [OpenHands issue #12279](https://github.com/OpenHands/OpenHands/issues/12279)
- [OpenHands PR #14692](https://github.com/OpenHands/OpenHands/pull/14692)
- [OpenHands contributing guide](https://github.com/OpenHands/OpenHands/blob/main/CONTRIBUTING.md)
- [Issue-linked V1 conversation WebSocket file](https://github.com/OpenHands/OpenHands/blob/main/frontend/src/contexts/conversation-websocket-context.tsx)
- [Issue-linked V0 WebSocket provider reference](https://github.com/OpenHands/OpenHands/blob/main/frontend/src/context/ws-client-provider.tsx)
- [Issue-linked send-message hook](https://github.com/OpenHands/OpenHands/blob/main/frontend/src/hooks/use-send-message.ts)
- [Issue-linked frontend test file](https://github.com/OpenHands/OpenHands/blob/main/frontend/__tests__/conversation-websocket-handler.test.tsx)
