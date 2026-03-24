---
id: b4w36sygpqprksyohrndv0h
title: Architecture
desc: ''
updated: 1774308901077
created: 1774307965611
---

The Semantic Flow API is a public, implementation-neutral contract. Weave is one implementation of that contract, not the contract itself.

The API should be resource-oriented and HTTP-first. Long-running work should be modeled as first-class `Job` resources rather than treated as an exceptional side channel.

## Recommendation

- Use OpenAPI as the canonical public contract for resources and state transitions.
- Standardize long-running work around durable `Job` resources:
  - create work
  - inspect status
  - discover results
  - represent failure and cancellation consistently
- Standardize public event channels as a companion contract when live progress matters.
- Use AsyncAPI for the event surface if that event surface is meant to be normative and interoperable across implementations.

## Architectural Posture

- OpenAPI remains primary.
- AsyncAPI complements OpenAPI; it should not replace the main resource contract.
- Polling against `Job` resources is the universal baseline.
- Live progress is an enhancement over the canonical resource model, not a separate source of truth.

## Events

Public events are worth standardizing if third-party clients are expected to track long-running work well. A reasonable initial event surface is:

- `job.accepted`
- `job.progress`
- `job.succeeded`
- `job.failed`
- `job.cancelled`
- optionally `resource.changed`

Implementation-specific internal events should stay out of the public contract, especially daemon queue internals, lock-manager chatter, and low-level filesystem watch noise.

## Clients

SSE is a good default live-progress transport for HTTP clients, including CLI clients. CLI support does not require a separate event architecture.

The CLI may support both:

- remote HTTP mode against a daemon implementation
- offline local or in-process execution mode against shared logic, with no daemon and no network required

Offline CLI support should be treated as a first-class execution mode, not merely a fallback. The same semantic operations should be available in both modes as closely as practical.

That duality is a client and implementation concern, not a reason to weaken the public Semantic Flow API contract.
