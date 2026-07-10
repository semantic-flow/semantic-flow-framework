---
id: s57t4fyvoq43nknu3yrh01g
title: Integrity
desc: ''
updated: 1783659521379
created: 1783659397616
---

## From stagecraft...

Since in SF state IRIs are supposed to be immutable-ish, and we intend to publish content hashes for DigitalArtifacts, maybe someday SF could do the commit hash's job?
That's a load-bearing "someday" — and I think it's architecturally sound, because the manifest you saw is already half the shape. Let me take it apart properly, then I'll board the direction so it doesn't evaporate.

What the commit hash actually does for us — four distinct jobs:

1. Single-artifact integrity — "these bytes are the ones." Published content hashes on DigitalArtifact versions do this job outright. Planned, easy, done.
2. Bundle integrity — the checkpoint's evidence isn't one file, it's the joins: state + manifest entry + authority chain agreeing. Git covers this with its Merkle tree (one hash over the whole tree). SF replicates it the moment manifest entries record the content hashes of everything they reference — then the manifest version's own published hash becomes a Merkle root over the bundle. The manifest already references the state version and the checkpoint referent; adding recorded hashes turns it from a ledger line into a tamper-evident one. This is the biggest single step, and it's small.
3. History integrity — git chains parent hashes, so rewriting the past is loud. SF's artifact histories have ordinals (s0001…s0008), which give ordering but not tamper-evidence. The fix is hash-chaining: each version record carries the previous version's hash. That makes every SF dataset an append-only log in its own right — "immutable-ish" hardens into immutable-or-caught.
4. The root of trust — any Merkle system needs an unforgeable tip. Git gets this socially (remotes, clones, protected branches). SF has something better available in-platform: the authority already signs — commit certificates are the natural tip-attestation, and the checkpoint act already mints one. And at the boundary where artifacts leave our trust domain, this converges exactly onto your parked Part-6 note: the cross-trust-domain anchor (token, transparency log) was always the missing outer link, and it stays parked — but now it has a precise job description.


What it buys, concretely: identity and integrity collapse into one plane — the semantic plane becomes self-verifying, and the weakness I named two turns ago ("a SHA is per-repo; it doesn't travel as identity") dissolves. Meshes become substrate-portable: re-host on S3, IPFS, anything — verification rides in the data. Portable character provenance stops needing git at the receiving table; the evidence chain travels as RDF with its own roots. Publish-as-seed redaction can re-root a filtered bundle honestly. And the 2109 class of bug stops being guarded against and becomes inexpressible: a dependency pin would BE a content hash, and loaded bytes either hash to it or don't — no "a named ref happens to resolve to the same commit" ambiguity left to adjudicate.

Three cautions, honestly: First, git stays as the working substrate — editing, collaboration, transport, GC; this proposal is about which plane carries truth, not about writing a second VCS. Second, do it as a ladder, each rung useful alone: hashes on artifacts → hashes in manifest entries → chained histories → certificate-signed tips. Third, the migration path is "both agree": the 2109-style verification tooling should check SF hashes and git corroboration together first, and only when SF-alone has run clean for a while does git demote to plumbing. (Plus the small hygiene note: record the hash algorithm in the data — algorithms rotate.)

One more convergence worth savoring: the weave already computes hashes at flush time for input-snapshot verification — so "publish them into the manifest" is asking the tooling to write down what it already knows. The weave team's stated target workload is exactly this mesh; this might be their next feature request from us.

Boarding it now as a parked direction with its wake triggers.