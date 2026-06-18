This building block defines a JSON input format for describing the OGC knowledge graph hierarchy of linked
data resources used by [Prez](https://github.com/RDFLib/prez), and provides the semantic uplift pipeline to
convert it to RDF. It is a variant of the [default Prez hierarchy](bblocks://ogc.prez.hierarchy.default)
building block, generalized to support the structure of the OGC definitions server: a distinct top-level
catalog type, and Catalogs that can be nested to any depth.

## Structure

The top-level input is either a single TopCatalog object, or an array mixing TopCatalog objects with
[namespace prefix declarations](bblocks://ogc.prez.prefix). Prefix declarations have no `id` — they
uplift to standalone blank nodes, unrelated to the catalogs or to each other, used by Prez to build
its global CURIE map.

A TopCatalog (`dcat:TopCatalog`) sits at the root of the hierarchy. Its `items` (and, recursively, the
`items` of any nested Catalog) can be any mix of:

| Type | JSON discriminator | RDF type |
|---|---|---|
| Catalog | `type: Catalog` | `dcat:Catalog` |
| Concept Scheme | `type: ConceptScheme` | `skos:ConceptScheme` |
| Collection | `type: Collection` | `skos:Collection` |
| Dataset | `type: Dataset` | `dcat:Dataset` |
| Resource | `type: Resource` | `dcat:Resource` |
| Ontology | `type: Ontology` | `owl:Ontology` |

Concept Schemes and Collections in turn contain **concepts** (`skos:Concept`).

A TopCatalog's or Catalog's items can also be plain IRI strings instead of embedded objects, for
resources that are already defined elsewhere — for example, in a different graph. A plain string
becomes a `dct:hasPart` link to that IRI, without asserting any type or label locally; this allows
declaring standalone catalogs whose concept schemes, datasets, etc. live elsewhere.

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

Unlike the default hierarchy, `type` is **required on every entry**, including Concept Schemes and
Concepts. This building block has more entry types than the default hierarchy, and several of them
(TopCatalog vs. Catalog, in particular) share the exact same shape (`id`, `name`, `items`), so there is
no reliable structural cue to infer the type automatically — an explicit `type` keeps the model
unambiguous and avoids a type-inference step in the uplift pipeline.

The `name` field (mapped to `skos:prefLabel`) supports both plain strings and language-tagged objects, e.g.:

```json
"name": { "en": "Habitat Types", "es": "Tipos de Hábitat" }
```

## Semantic uplift

The following steps are applied after JSON-LD conversion:

1. **SPARQL UPDATE** — entails the inverse of any asserted `skos:broader` / `skos:narrower` relation.
2. **SPARQL UPDATE** — propagates `skos:inScheme` along `skos:narrower` chains, so that concepts nested
   only inline (not listed under a Concept Scheme's `concepts`) are still anchored to their scheme.
3. **SPARQL UPDATE** — adds `skos:topConceptOf` / `skos:hasTopConcept` links between root concepts (those
   without a `skos:broader`) and their concept schemes, enabling richer hierarchy display in Prez.

## Vocabularies

| Prefix | Namespace |
|---|---|
| `dcat` | `http://www.w3.org/ns/dcat#` |
| `dct` | `http://purl.org/dc/terms/` |
| `skos` | `http://www.w3.org/2004/02/skos/core#` |
| `owl` | `http://www.w3.org/2002/07/owl#` |
| `vann` | `http://purl.org/vocab/vann/` |
