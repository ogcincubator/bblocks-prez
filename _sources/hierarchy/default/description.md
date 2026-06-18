This building block defines a JSON input format for describing the default [Prez](https://github.com/RDFLib/prez) hierarchy of linked data resources, and provides the semantic uplift pipeline to convert it to RDF.

## Structure

The top-level input is either a single catalog object, or an array mixing catalog objects with
[namespace prefix declarations](bblocks://ogc.prez.prefix). Prefix declarations have no `id` — they
uplift to standalone blank nodes, unrelated to the catalogs or to each other, used by Prez to build
its global CURIE map. A catalog contains **second-level entries** — any mix of:

| Type | JSON discriminator | RDF type |
|---|---|---|
| Concept Scheme | `concepts` array present | `skos:ConceptScheme` |
| Collection | `type: Collection` (required) | `skos:Collection` |
| Dataset | `type: Dataset` (required) | `dcat:Dataset` |
| Resource | `type: Resource` (required) | `dcat:Resource` |

Concept Schemes and Collections in turn contain **concepts** (`skos:Concept`).

A catalog's items can also be plain IRI strings instead of embedded objects, for resources that are
already defined elsewhere — for example, in a different graph. A plain string becomes a `dct:hasPart`
link to that IRI, without asserting any type or label locally; this allows declaring standalone
catalogs whose concept schemes, datasets, etc. live elsewhere.

Concepts can be arranged into a `skos:broader` / `skos:narrower` hierarchy:

- `broader` is an array of concept ids (strings) referencing already-declared concepts.
- `narrower` is an array mixing concept ids (strings) and/or **embedded Concept objects** — allowing a
  whole sub-tree to be declared inline under its parent, instead of being listed flatly under the
  Concept Scheme's `concepts`.

Only one direction needs to be asserted per relation; the inverse is entailed during semantic uplift.
A concept nested only inside another concept's `narrower` (and not also listed directly under
`concepts`) is automatically anchored to the same Concept Scheme as its ancestor.

Only concepts with no `skos:broader` (i.e. the roots of the hierarchy) are marked as
`skos:topConceptOf` their scheme — concepts that are narrower of another concept are not top concepts.

The `type` field is optional for Catalogs, Concept Schemes, and Concepts — it is inferred automatically during semantic uplift from the structural properties of each object.

The `name` field (mapped to `skos:prefLabel`) supports both plain strings and language-tagged objects, e.g.:

```json
"name": { "en": "Habitat Types", "es": "Tipos de Hábitat" }
```

## Semantic uplift

The following steps are applied after JSON-LD conversion:

1. **jq transform** — infers missing `type` values based on object structure before JSON-LD expansion.
2. **SPARQL UPDATE** — entails the inverse of any asserted `skos:broader` / `skos:narrower` relation.
3. **SPARQL UPDATE** — propagates `skos:inScheme` along `skos:narrower` chains, so that concepts nested
   only inline (not listed under a Concept Scheme's `concepts`) are still anchored to their scheme.
4. **SPARQL UPDATE** — adds `skos:topConceptOf` / `skos:hasTopConcept` links between root concepts (those
   without a `skos:broader`) and their concept schemes, enabling richer hierarchy display in Prez.

## Vocabularies

| Prefix | Namespace |
|---|---|
| `dcat` | `http://www.w3.org/ns/dcat#` |
| `dct` | `http://purl.org/dc/terms/` |
| `skos` | `http://www.w3.org/2004/02/skos/core#` |
| `vann` | `http://purl.org/vocab/vann/` |
