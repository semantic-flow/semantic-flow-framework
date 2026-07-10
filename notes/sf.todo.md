---
id: l59hvxem2488war5j981yca
title: SF Todo
desc: ''
updated: 1782141962126
created: 1731041743071
---

## Immediately




## Eventually

- Consider a direct-content `version` API (submit bytes + target `DigitalArtifact` IRI + optional metadata, Weave appends a `HistoricalState` without a working-file round-trip). Deferred while consumers keep a filesystem-backed mesh; see [[sf.product-ideas.api]].

- Decide whether `mesh-content/` should live under `_mesh/` instead of as a top-level sibling. The likely direction is to keep mesh-owned page-source/support content inside the mesh support surface, perhaps as `_mesh/content/` or `_mesh/_content/`, so top-level paths stay focused on public identifiers and working payloads. Check the impact on page-source examples, `targetLocalRelativePath` conventions, root-page customization fixtures, and sidecar mesh layouts before changing the convention.

- late answers on
  - https://stackoverflow.com/questions/46705136/how-to-make-an-ontology-public-accessible
  - https://www.reddit.com/r/semanticweb/comments/1rxdk1a/comment/obbqgoa/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button
    - "I can tell that the biggest pain points are definitively URIs. As someone already pointed out, URIs should not be bound to location. For one, things like the used protocol (e.g. http, https) are quite irrelevant for identifying a concept semantically and second it creates the need to come up with a URI scheme that in 90% of the cases will not be resolvable because the infra does not exist. So it effectively becomes a very bulky id."
