---
id: n8x2m5q1v7r4k9p3t6h0bca
title: 2026 08 21 Content Digest
desc: 'Portable content-digest representation, bearer, expectation, observation, and verification behavior'
created: 1787371200000
---

## Purpose

This specification defines portable Semantic Flow content-digest behavior across RDF assertions, artifact resolution, validation, and implementations such as Weave.

## Supported Representation

The next Semantic Flow release supports SHA-256 content digests in exactly this lexical form:

`sha256:<64 lowercase hexadecimal digits>`

The value is an `xsd:string`. Uppercase hexadecimal, missing or different method tokens, abbreviated values, and non-hexadecimal characters do not conform.

`ContentDigestMethod` is an open controlled vocabulary. `contentDigestMethod_sha256`, with `contentDigestMethodToken "sha256"`, is the only method supported by this release. A later release may add methods by adding vocabulary members, grammar, implementation support, and tests together.

## Standing Digest Claims

`hasContentDigest` is a standing claim about the subject's own represented or retrievable bytes. Its domain is `ContentDigestBearer`.

SFLO directly classifies two bearer types:

- `ArtifactManifestation`: the digest claims the exact byte sequence represented by that manifestation
- `LocatedFile`: the digest is an independently checkable claim about bytes retrieved through that file identity or location

An `ArtifactManifestation` denotes one exact byte sequence. Formatting, line-ending, compression, packaging, canonicalization, or serialization changes that alter bytes create a different manifestation. Multiple located files may provide one manifestation only as byte-identical replicas.

When a manifestation and one of its `locatedFileForManifestation` values both declare a digest for the same method, the values must agree. A mismatch is inconsistent data and fails validation.

A bearer may carry values for multiple supported methods in a future release, but it must not carry multiple distinct digest values for the same method. The same uniqueness rule applies to observed digest evidence on one observation.

`DigitalArtifact`, `ArtifactHistory`, and `HistoricalState` do not become digest bearers merely through their existing types because they may span multiple representations. A `RepositorySourceLocator` identifies repository coordinates rather than the content resolved from them and must not carry `hasContentDigest`.

Downstream vocabularies may define narrower `ContentDigestBearer` subclasses when the resource identifies one determinate or retrievable byte stream. Using `hasContentDigest` otherwise infers only `ContentDigestBearer`; it does not infer `ArtifactManifestation` or `LocatedFile` specifically.

## Expected and Observed Digests

`expectsContentDigest` is a pre-resolution requirement on `ArtifactResolutionSpec`. It comes from a caller, authored policy, or durable source binding before resolution begins.

`observedContentDigest` is evidence computed from concrete bytes during one `ArtifactResolutionObservation`.

An implementation must not copy a digest it has just observed into `expectsContentDigest` after the fact. When an expected and observed digest exist for the same linked resolution and differ for the same method, verification fails. The implementation must not use the bytes as a successful result or persist the mismatch as though it were a successful observation. Modeling failed-attempt provenance is outside this first contract.

Repository-backed source bindings put deliberately supplied digest requirements on the binding with `expectsContentDigest`. They put computed digest evidence on a linked observation with `observedContentDigest`. Exact and floating repository locator nodes carry none of the three digest properties.

When a file-backed integration computes an observation, its `observedArtifactResolutionSpec` records the concrete local relative path actually read. A runtime should add a target located file, manifestation, state, or other exact coordinate when it genuinely knows that identity; it must not invent one merely to shorten a later query.

## Authored File and Published Mirror Example

One exact manifestation may be available through an authored source location and a published mirror when both return identical bytes:

```turtle
@prefix sflo: <https://semantic-flow.github.io/sflo/ontology/> .
@prefix schema: <https://schema.org/> .

<manifestation/source-markdown> a sflo:ArtifactManifestation ;
  sflo:hasContentDigest "sha256:aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" ;
  sflo:locatedFileForManifestation <file/authored>, <file/published> .

<file/authored> a sflo:LocatedFile ;
  schema:contentUrl "sources/example.md" ;
  sflo:hasContentDigest "sha256:aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" .

<file/published> a sflo:LocatedFile ;
  schema:contentUrl "published/example.md" ;
  sflo:hasContentDigest "sha256:aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa" .
```

If either file returns different bytes, it no longer truthfully provides this manifestation. A different-byte published file requires a different `ArtifactManifestation`.

Existing data that places a canonical digest on a `LocatedFile` remains valid. Moving or copying that assertion to the manifestation is optional unless a downstream profile specifically requires manifestation-level identity.

## Joining Standing Claims to Observations

No direct observation-to-bearer property is required. The concrete `observedArtifactResolutionSpec` can target a located file or manifestation, and SPARQL property paths provide the comparison:

```sparql
PREFIX sflo: <https://semantic-flow.github.io/sflo/ontology/>

SELECT ?bearer ?claimedDigest ?observedDigest WHERE {
  ?observation
    sflo:observedContentDigest ?observedDigest ;
    sflo:observedArtifactResolutionSpec/
      (sflo:targetLocatedFile|sflo:targetManifestation) ?bearer .
  ?bearer sflo:hasContentDigest ?claimedDigest .
}
```

An implementation recording an observation should put the most concrete coordinates it actually knows in `observedArtifactResolutionSpec`. It should not mint a target relation merely to make a query shorter.

## Validation and Runtime Responsibilities

- SHACL applies the exact supported grammar to `hasContentDigest`, `expectsContentDigest`, and `observedContentDigest` at violation severity.
- SFLO validates agreement between manifestation and located-file standing claims when both are present.
- Runtimes validate expected/observed agreement before successful use or persistence. SHACL deliberately does not compare a spec's current expectation with every appendable historical observation, because a later expectation update must not invalidate honest earlier evidence.
- Runtimes compute SHA-256 over the exact resolved byte sequence without text normalization.
- Runtimes reject unsupported or noncanonical expected values rather than silently changing the persisted wire value.
- Floating repository source locators remain prohibited from carrying content digests.

## Non-Goals

- Structured digest nodes, SPDX checksum structures, signatures, or attestations beyond the existing observation relator.
- Additional digest methods.
- Canonical hashing of RDF graphs, repository trees, directories, archives, or abstract historical states.
- Treating a computed observation as a retroactive expectation.
- Requiring all located files to carry digests.
