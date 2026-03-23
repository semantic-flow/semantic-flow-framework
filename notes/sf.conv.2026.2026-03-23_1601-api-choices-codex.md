---
id: api-choices-1hzxvy
title: 'api choices'
desc: ''
created: 1774306876465
updated: 1774306876465
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

# djradon_2026-03-23_0641_37

The "Semantic Flow API Spec" is a public contract. Weave daemon is just an implementation of that spec.

# gpt-5.4_2026-03-23_0641_54

That boundary is cleaner: the Semantic Flow API Spec is the product contract, and the Weave daemon is one implementation of it.

With that framing, I’d refine the options to these:

- `OpenAPI-only public spec`
  - Model long-running work as first-class `Job` or `Operation` resources.
  - `POST` starts work, `202 Accepted` returns a job URL, `GET` reads status/results.
- `OpenAPI + SSE`
  - Same public resource model, plus a standard event stream for progress.
  - This is still basically an OpenAPI-centered API, not an AsyncAPI-first one.
- `OpenAPI + AsyncAPI companion`
  - OpenAPI defines the canonical contract.
  - AsyncAPI separately defines progress, completion, fs-change, and subscription channels.
  - This is the strongest hybrid if events are important enough to deserve a real public contract.
- `Protocol-neutral core model + bindings`
  - Define the semantic contract at the ontology/resource level, then publish OpenAPI and maybe AsyncAPI bindings.
  - Architecturally elegant, but probably too heavy unless you expect multiple non-HTTP implementations soon.

So yes, you were missing one good option: `OpenAPI with explicit operation resources`, where “asynchronous” is a behavior of the resource model, not a separate API style.

My current recommendation would be:
- Public `Semantic Flow API Spec`: OpenAPI-primary
- Optional companion event contract: AsyncAPI only if you want event channels standardized publicly
- Weave daemon: one implementation, possibly with a separate private/admin API that is not part of the public Semantic Flow spec

The main reason is that Semantic Flow still sounds resource-centric and dereferenceable first, event-centric second. AsyncAPI becomes compelling when events are the core domain object, not just the way clients observe long-running work.

# djradon_2026-03-23_1553_15

It's a public API, why wouldn't we want event channels standardized publicly?

# gpt-5.4_2026-03-23_1553_35

You probably do want them standardized publicly. The part I’d push on is not public vs private, but how central they should be and what exactly gets standardized.

If third-party clients are expected to track long-running work well, a public event contract is valuable because otherwise every implementation invents its own progress stream semantics. That weakens interoperability fast. So I’d frame it like this:

- Standardize `Job` or `Operation` resources publicly no matter what.
- Standardize public event channels too, if live progress is part of the intended client experience.
- Keep the resource model canonical, and make events a companion contract over it.

The reason not to rush into “everything AsyncAPI-first” is that events are usually not the whole API surface here. Semantic Flow still looks resource-centric:
- clients create or mutate resources
- server starts long-running work
- work is represented as a durable resource
- events are how clients observe state changes

That usually argues for:
- OpenAPI for the main resource contract
- AsyncAPI for the event contract, if you want event behavior formally standardized

What I would avoid is leaving events unspecified or hand-wavy. If they matter, specify them publicly. But I would also avoid letting transient implementation details leak into the public event model.

A good public split is:

- Canonical resource contract:
  - `POST /jobs`
  - `GET /jobs/{id}`
  - terminal states, result links, error model, cancellation semantics
- Companion event contract:
  - `job.accepted`
  - `job.progress`
  - `job.completed`
  - `job.failed`
  - maybe `resource.changed`

What I would not standardize early:
- daemon-internal queue events
- lock manager chatter
- low-level filesystem watch noise
- transport-specific operational details that may vary by implementation

So the short answer is: yes, if event channels are part of the intended public developer experience, they should be standardized publicly. My only caution is that this still doesn’t mean AsyncAPI should replace the main resource contract. It more likely means “OpenAPI + AsyncAPI companion” rather than “AsyncAPI-based API.”

If you want, I can turn that into a crisp recommendation matrix for the architecture note: `OpenAPI only` vs `OpenAPI + SSE` vs `OpenAPI + AsyncAPI companion` with a clear recommendation.
