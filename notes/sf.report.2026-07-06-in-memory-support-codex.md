---
id: ivutbrv54tm3kvqku8k8td2
title: 2026 07 06 in Memory Support Codex
desc: ''
updated: 1783397345922
created: 1783397331855
---

You’re right. That comment muddles two different things.

What I meant was “do not mint a different state than the caller intended,” but saying “no ordinal allocation” is wrong if every `HistoricalState` is ordinaled. Better split:

- `stateOrdinal`: always present.
- `stateSegment`: optional human/path segment.
- `stateIri`: either exact, or derived from `{historyIri, stateSegment ?? ordinalSegment(stateOrdinal)}`.
- ordinal advancement: either performed by Weave or verified against caller-supplied pre-advanced state.

I’d reshape it like this:

```ts
type HistoricalStateTarget =
  | {
      kind: "next";
      stateSegment?: string;      // e.g. "v0.3.0"; otherwise "_s0042"
      stateOrdinal?: number;      // optional assertion; fail if not next
    }
  | {
      kind: "exact";
      stateIri: string;
      stateOrdinal: number;       // required
      requireNextOrdinal?: true;  // default true for flush paths
    };
```

For Stagecraft flush-all, I think the right default is:

```ts
mode: "realizePreadvancedState"
state: {
  kind: "exact",
  stateIri,
  stateOrdinal,
  requireNextOrdinal: true
}
```

That says: “memory already advanced the history; disk must faithfully realize that same state.” Weave checks the target history’s `nextStateOrdinal`; if it is not equal to `stateOrdinal`, fail closed. Then it writes `stateOrdinal`, updates `latestHistoricalState`, and sets `nextStateOrdinal = stateOrdinal + 1`.

For file-based `weave version`, the convenience path can use:

```ts
mode: "advanceAndRealizeState"
state: {
  kind: "next",
  stateSegment
}
```

If no `stateSegment` is supplied, Weave derives `_sNNNN` from the next ordinal. If a named segment is supplied, the state still gets the next ordinal; the segment is just the path/IRI token.

So yes: passing an ordinal is useful, but I’d make it a consistency assertion, not a request to skip the normal ordinal stream. That preserves the “one ordinal space, no reminting” rule nicely.