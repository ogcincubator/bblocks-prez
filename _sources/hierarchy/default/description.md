This building block defines a JSON input format for describing the default [Prez](https://github.com/RDFLib/prez) hierarchy of linked data resources, and provides the semantic uplift pipeline to convert it to RDF.

## Structure

The top-level input is either a single catalog object or a wrapper object with a `base` URL and a `catalogs` array. A catalog contains **second-level entries** — any mix of:

| Type | JSON discriminator | RDF type |
|---|---|---|
| Concept Scheme | `concepts` array present | `skos:ConceptScheme` |
| Collection | `type: Collection` (required) | `skos:Collection` |
| Dataset | `type: Dataset` (required) | `dcat:Dataset` |
| Resource | `type: Resource` (required) | `dcat:Resource` |

Concept Schemes and Collections in turn contain **concepts** (`skos:Concept`).

The `type` field is optional for Catalogs, Concept Schemes, and Concepts — it is inferred automatically during semantic uplift from the structural properties of each object.

The `name` field (mapped to `skos:prefLabel`) supports both plain strings and language-tagged objects, e.g.:

```json
"name": { "en": "Habitat Types", "es": "Tipos de Hábitat" }
```

## Semantic uplift

Two steps are applied after JSON-LD conversion:

1. **jq transform** — infers missing `type` values based on object structure before JSON-LD expansion.
2. **SPARQL UPDATE** — adds `skos:topConceptOf` / `skos:hasTopConcept` links between concepts and their concept schemes, enabling richer hierarchy display in Prez.

## Vocabularies

| Prefix | Namespace |
|---|---|
| `dcat` | `http://www.w3.org/ns/dcat#` |
| `dct` | `http://purl.org/dc/terms/` |
| `skos` | `http://www.w3.org/2004/02/skos/core#` |
