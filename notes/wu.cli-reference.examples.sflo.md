---
id: ewr0vs8qs6fpkrs63y7tzll
title: SFLO CLI Examples
desc: ''
updated: 1779030765340
created: 1779030765340
---

## Purpose

This note records the command sequence used while dogfooding Weave against the SFLO ontology repository and a branch-published `gh-pages` mesh. The examples assume a Weave checkout with dependency-local sibling checkouts for the SFLO source repository and the SFLO publication worktree.

The source checkout stays on `main`. The publication checkout is an unborn or disposable `gh-pages` worktree. The ontology and SHACL payload artifacts use `release` as the artifact-history segment and `v0.1.0` as the state segment.

## Step 1: Set Shell Variables

Use variables so the command sequence is replayable from the Weave checkout without repeating long local paths. `WEAVE_LOG_DIR` keeps local runtime logs out of the publication worktree.

```sh
export WEAVE_ROOT=/home/djradon/hub/semantic-flow/weave
export WEAVE_CLI="$WEAVE_ROOT/src/main.ts"
export SFLO_SRC="$WEAVE_ROOT/dependencies/github.com/semantic-flow/sflo"
export SFLO_PUB="$WEAVE_ROOT/dependencies/github.com/semantic-flow/sflo-gh-pages"
export SFLO_SOURCE_REPO='https://github.com/semantic-flow/sflo.git'
export WEAVE_LOG_DIR=/tmp/weave-logs

mkdir -p "$WEAVE_LOG_DIR"
cd "$WEAVE_ROOT"
```

## Step 2: Reset The Disposable Publication Worktree

This clears generated publication output while preserving `welcome.ttl`. Use this only when the `gh-pages` worktree has no committed publication history you need to keep.

```sh
tmp_welcome="$(mktemp)"
cp "$SFLO_PUB/welcome.ttl" "$tmp_welcome"

find "$SFLO_PUB" -mindepth 1 -maxdepth 1 ! -name .git -exec rm -rf {} +

cp "$tmp_welcome" "$SFLO_PUB/welcome.ttl"
rm "$tmp_welcome"

git -C "$SFLO_PUB" status --short --branch
```

## Step 3: Create The Publication Mesh

Create the branch-published mesh support artifacts at the publication root. Root publication meshes do not need `_mesh/_config/config.ttl`.

```sh
deno run -A "$WEAVE_CLI" mesh create \
  --workspace "$SFLO_PUB" \
  --mesh-base 'https://semantic-flow.github.io/sflo/'
```

## Step 4: Integrate The Root Welcome Page

Integrate `welcome.ttl` as the root designator `/`. In this sequence, `/` identifies the site's welcome page rather than a separate "semantic site" resource.

```sh
deno run -A "$WEAVE_CLI" integrate "$SFLO_PUB/welcome.ttl" / \
  --mesh-root "$SFLO_PUB"
```

## Step 5: Weave The Root Welcome Page

Generate the root ResourcePage with `main` as the history segment. Let Weave use the default state segment for the welcome page instead of naming a version-like state manually.

```sh
deno run -A "$WEAVE_CLI" \
  --mesh-root "$SFLO_PUB" \
  --target 'designatorPath=/,historySegment=main'
```

## Step 6: Materialize The Core Ontology

Prepare the publication root and materialize the core ontology source file as the `ontology` payload artifact. `--allow-dirty-publish-root` is used because the previous steps intentionally left generated, uncommitted files in the publication worktree.

Before publication, the source ontology metadata should consistently describe `v0.1.0`, and `owl:versionIRI` should point at raw bytes for the matching release tag. The version/HistoricalState resource can use `schema:contentUrl` for the preferred mesh-served Turtle URL and `sflo:hasManifestation` for the lightweight manifestation node whose `dcat:downloadURL` is the same mesh-served Turtle URL. The generated inventory carries the richer `ArtifactManifestation`, `LocatedFile`, and `locatedFileForManifestation` graph. Commit and tag the SFLO source first, then use the tag as `--source-ref`; do not record `main` for release-versioned bytes.

```sh
deno run -A "$WEAVE_CLI" prepare gh-pages \
  --source-root "$SFLO_SRC" \
  --publish-root "$SFLO_PUB" \
  --mesh-base 'https://semantic-flow.github.io/sflo/' \
  --source-path semantic-flow-core-ontology.ttl \
  --target-path semantic-flow-core-ontology.ttl \
  --designator-path ontology \
  --payload-history-segment release \
  --payload-state-segment v0.1.0 \
  --payload-manifestation-segment ttl \
  --source-repository-url "$SFLO_SOURCE_REPO" \
  --source-ref v0.1.0 \
  --allow-dirty-publish-root
```

## Step 7: Materialize The Config Ontology

Materialize the config ontology source file as the `config` payload artifact, using the same release/state/manifestation naming pattern.

```sh
deno run -A "$WEAVE_CLI" prepare gh-pages \
  --source-root "$SFLO_SRC" \
  --publish-root "$SFLO_PUB" \
  --mesh-base 'https://semantic-flow.github.io/sflo/' \
  --source-path semantic-flow-config-ontology.ttl \
  --target-path semantic-flow-config-ontology.ttl \
  --designator-path config \
  --payload-history-segment release \
  --payload-state-segment v0.1.0 \
  --payload-manifestation-segment ttl \
  --source-repository-url "$SFLO_SOURCE_REPO" \
  --source-ref v0.1.0 \
  --allow-dirty-publish-root
```

## Step 8: Materialize The Core SHACL Shapes

Materialize the core SHACL source file as the `ontology/shacl` payload artifact.

```sh
deno run -A "$WEAVE_CLI" prepare gh-pages \
  --source-root "$SFLO_SRC" \
  --publish-root "$SFLO_PUB" \
  --mesh-base 'https://semantic-flow.github.io/sflo/' \
  --source-path semantic-flow-core-shacl.ttl \
  --target-path semantic-flow-core-shacl.ttl \
  --designator-path ontology/shacl \
  --payload-history-segment release \
  --payload-state-segment v0.1.0 \
  --payload-manifestation-segment ttl \
  --source-repository-url "$SFLO_SOURCE_REPO" \
  --source-ref v0.1.0 \
  --allow-dirty-publish-root
```

## Step 9: Inspect The Generated Publication Worktree

Confirm the root welcome page uses `main/_s0001`, and the ontology and SHACL payloads use `release/v0.1.0/ttl`.

```sh
find "$SFLO_PUB" \
  -path '*/release/v0.1.0/ttl/*.ttl' \
  -o -path '*/_s0001/ttl/*.ttl' \
  | sort

git -C "$SFLO_PUB" status --short --branch
```
