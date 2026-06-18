# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

An [OGC Building Blocks](https://ogcincubator.github.io/bblocks-docs/) register (based on the
`bblock-template`) for [Prez](https://github.com/RDFLib/prez), a read-only linked data API/browser
built on SPARQL. Building blocks here define JSON input formats and semantic uplift pipelines that
transform structured JSON documents into RDF graphs ready for publication via Prez, using DCAT and
SKOS vocabularies, JSON Schema, SHACL, and jq/SPARQL-based uplift.

There is no application source code to build/lint/test in the traditional sense — the "build" is the
bblocks postprocessing pipeline (a Docker tool maintained outside this repo) that validates and compiles
sources into ready-to-use artifacts.

## Commands

- `./build.sh` — runs the postprocessing pipeline via Docker (`ghcr.io/opengeospatial/bblocks-postprocess`)
  against the sources in `_sources/`, writing output to `build-local/` (gitignored). This is the
  equivalent of the CI validation job and is the way to test changes locally.
  - Honors an optional `.volumes` file (one mount per line; relative paths are resolved against the repo root)
    for mounting additional volumes into the container.
- `./view.sh` — runs `ghcr.io/ogcincubator/bblocks-viewer` via Docker on port 9090 to preview the compiled
  register. Run `./build.sh` first (or delete `build-local` to preview the committed `build/` output instead).
- Both scripts require Docker locally.
- CI (`.github/workflows/process-bblocks.yml`) calls the same shared `validate-and-process.yml` workflow on
  push to `master`/`main`, publishing to GitHub Pages.

## Architecture

- `_sources/<path>/bblock.json` — metadata for a building block (name, abstract, status, version, tags,
  itemClass, register, etc.). The building block's identifier is `<identifier-prefix><dot-separated path>`
  (prefix set in `bblocks-config.yaml`, currently `ogc.prez.`). Directory nesting becomes part of the
  identifier and the generated docs path.
- `_sources/<path>/schema.yaml` (or `.json`) — JSON Schema for the building block. Can `$ref` other building
  blocks' schemas; `$_ROOT_/` refers to the root of the central OGC Building Blocks tree.
- `_sources/<path>/context.jsonld` — JSON-LD context performing "semantic annotation": maps schema properties
  to RDF predicates, enabling the schema's instance data to be parsed as RDF.
- `_sources/<path>/shapes.shacl` (Turtle) — SHACL shapes used to validate uplifted RDF graphs.
- `_sources/<path>/semantic-uplift.yaml` — defines the jq/SPARQL-based uplift pipeline from validated JSON to RDF.
- `_sources/<path>/examples.yaml` + `_sources/<path>/examples/*.json` — example instances; referenced via `ref:`
  to keep `examples.yaml` itself light. Examples are validated, uplifted, and rendered in the docs.
- `_sources/<path>/description.md` — human-readable Markdown description (relative links/images are resolved
  to full URLs on publish).
- `_sources/<path>/tests/` — `*.json`/`*.jsonld`/`*.ttl` test fixtures validated against the JSON Schema and/or
  SHACL shapes; filenames ending in `-fail` are expected to fail validation (negative tests).
- `build/` — committed, **never edited by hand** — the last known-good compiled output, kept for reference/reuse.
- `build-local/` — gitignored scratch output from `./build.sh`, used for local preview/testing.
- `bblocks-config.yaml` — register-level metadata: `name`, `abstract`, `description`, `identifier-prefix`,
  `imports` (other registers to resolve cross-repo `$ref`s/identifiers against — `default` aliases the main
  OGC register), and optional `sparql` push/query endpoint config.

Existing building blocks in `_sources/`:
- `hierarchy/default` — default Prez hierarchy building block.
- `prefix` — namespace prefix declaration (VANN-style) used for CURIE generation.

For detailed authoring guidance (schema/context/shapes/uplift/examples/tests conventions), consult the
`bblocks-authoring` skill rather than re-deriving conventions from scratch. If it isn't available in
this environment, fetch it (and other OGC-related skills) from
[https://ogcincubator.github.io/ogc-llm-skills/llms.txt](https://ogcincubator.github.io/ogc-llm-skills/llms.txt).