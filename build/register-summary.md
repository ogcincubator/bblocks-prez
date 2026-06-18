# Prez OGC Blocks

OGC Blocks for working with the [Prez](https://github.com/RDFLib/prez) application.


This collection provides OGC Building Blocks for working with [Prez](https://github.com/RDFLib/prez),
a read-only linked data API and browser built on SPARQL.

Building blocks in this collection define JSON input formats and semantic uplift pipelines that
transform structured JSON documents into RDF graphs ready for publication via Prez. They rely on
standard vocabularies — primarily DCAT and SKOS — and follow OGC patterns for schema definition,
validation (JSON Schema + SHACL), and semantic uplift (jq + SPARQL).


## Building Blocks

### `ogc.prez.prefix` — Namespace prefix declaration

**Type:** schema

Declares a preferred namespace prefix and the URI it expands to, for use in CURIE generation.

### `ogc.prez.hierarchy.default` — Prez default hierarchy

**Type:** schema

Helps create a default Prez hierarchy

