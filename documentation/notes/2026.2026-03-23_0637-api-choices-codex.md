---
id: api-choices-m91zse
title: 'api choices'
desc: ''
created: 1774273020569
updated: 1774273020569
participants: [djradon, codex.gpt-5.4]
conversationEventKinds: [message.assistant, message.user, tool.call, tool.result]
---

# djradon_2026-03-23_0634_56

api choices

Given the likely-long-running nature of several semantic-flow operations, I see two options:
- AsyncAPI-based API
- custom asynchronous OpenAPI

Am I missing any good options?

# gpt-5.4_2026-03-23_0635_02

I’m grounding on the repo’s architecture guidance first so the answer fits this project rather than giving a generic API take. I’ll read the product vision, the primary developer guidance note, and the current architecture-plior planning note, then I’ll compare the async API options against the kinds of long-running flows this codebase is aiming for.

I’ve got the current guidance in view. Next I’m checking the old API/service notes for the parts that matter here: how jobs, resource URLs, and daemon/API boundaries were previously framed, so I can tell you whether there’s a genuinely better third option or just cleaner ways to structure the two you already have.

The older notes reinforce one important thing: they already leaned toward a resource-oriented REST surface plus an explicit job model, with SSE for progress. I’m checking the host/service note now because that’s where the daemon coordination and lock/watch semantics live, and that affects whether AsyncAPI is a real fit or just a supporting spec.

I have enough context to answer usefully. I’m pulling exact line numbers for the current planning note and the older API note so I can anchor the recommendation in this repo’s existing architectural direction rather than hand-waving.

You’re missing one strong default: a plain resource-oriented HTTP API with first-class `Operation`/`Job` resources, optionally paired with an event stream.

Given Weave’s direction, I would not make AsyncAPI the primary contract. The repo keeps pointing toward a filesystem-shaped, dereferenceable API with daemon-backed jobs, not an event-native system: [product-vision.md](/home/djradon/hub/semantic-flow/weave/documentation/notes/product-vision.md#L9), [wd.task.2026.2026-03-20-architecture-planning.md](/home/djradon/hub/semantic-flow/weave/documentation/notes/wd.task.2026.2026-03-20-architecture-planning.md#L21), [product.plugins.sflo-api.md](/home/djradon/hub/semantic-flow/sflo/documentation/product.plugins.sflo-api.md#L10), [product.plugins.sflo-api.md](/home/djradon/hub/semantic-flow/sflo/documentation/product.plugins.sflo-api.md#L151).

The good options are:

- OpenAPI-primary, polling only. `POST` starts work, returns `202 Accepted` plus `Location: /jobs/{id}`, and clients poll `GET /jobs/{id}`.
- OpenAPI-primary plus SSE. Same job model, but add `/events` or `/jobs/{id}/events` for progress/completion. This looks like the best fit for Weave.
- Hybrid OpenAPI + AsyncAPI. Use OpenAPI for resources and job submission; use AsyncAPI only if the event surface itself becomes a real public contract.
- Internal RPC like gRPC/Connect behind an HTTP API. Viable for daemon internals, but probably the wrong public shape if API/site symmetry and dereferenceable resource URLs matter.

So the main pushback is: “AsyncAPI-based API” is probably the wrong primary framing. Long-running operations do not force you into AsyncAPI. They mostly force you into a solid job/operation model. If I were choosing today, I’d pick OpenAPI as the main spec, first-class job resources, and SSE as the initial observation channel; add AsyncAPI later only if the event model becomes substantial enough to deserve its own contract.
