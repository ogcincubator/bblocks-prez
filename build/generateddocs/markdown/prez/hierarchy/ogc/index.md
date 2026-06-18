
# Prez OGC hierarchy (Schema)

`ogc.prez.hierarchy.ogc` *v0.1*

Helps create the OGC knowledge graph hierarchy in Prez, with TopCatalogs, recursively nested Catalogs, and Ontologies.

[*Status*](http://www.opengis.net/def/status): Under development

## Description

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

## Examples

### Minimal TopCatalog
A single TopCatalog containing one concept scheme with two concepts.

#### json
```json
{
  "id": "https://example.org/cat/top",
  "name": "Example domain",
  "type": "TopCatalog",
  "items": [
    {
      "id": "https://example.org/cs/animals",
      "name": "Animal classification",
      "type": "ConceptScheme",
      "concepts": [
        { "id": "https://example.org/concept/mammal", "name": "Mammal", "type": "Concept" },
        { "id": "https://example.org/concept/bird", "name": "Bird", "type": "Concept" }
      ]
    }
  ]
}

```

#### jsonld
```jsonld
{
  "@context": "https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/hierarchy/ogc/context.jsonld",
  "id": "https://example.org/cat/top",
  "name": "Example domain",
  "type": "TopCatalog",
  "items": [
    {
      "id": "https://example.org/cs/animals",
      "name": "Animal classification",
      "type": "ConceptScheme",
      "concepts": [
        {
          "id": "https://example.org/concept/mammal",
          "name": "Mammal",
          "type": "Concept"
        },
        {
          "id": "https://example.org/concept/bird",
          "name": "Bird",
          "type": "Concept"
        }
      ]
    }
  ]
}
```

#### ttl
```ttl
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .

<https://example.org/cat/top> a dcat:TopCatalog ;
    dcterms:hasPart <https://example.org/cs/animals> ;
    skos:prefLabel "Example domain" .

<https://example.org/concept/bird> a skos:Concept ;
    skos:inScheme <https://example.org/cs/animals> ;
    skos:prefLabel "Bird" ;
    skos:topConceptOf <https://example.org/cs/animals> .

<https://example.org/concept/mammal> a skos:Concept ;
    skos:inScheme <https://example.org/cs/animals> ;
    skos:prefLabel "Mammal" ;
    skos:topConceptOf <https://example.org/cs/animals> .

<https://example.org/cs/animals> a skos:ConceptScheme ;
    skos:hasTopConcept <https://example.org/concept/bird>,
        <https://example.org/concept/mammal> ;
    skos:prefLabel "Animal classification" .


```


### Recursively nested catalogs
A TopCatalog containing a Catalog, which in turn contains another Catalog holding
a Dataset. Catalogs can be nested to any depth under a TopCatalog.

#### json
```json
{
  "id": "https://example.org/cat/top",
  "name": "Earth observation definitions",
  "type": "TopCatalog",
  "items": [
    {
      "id": "https://example.org/cat/climate",
      "name": "Climate",
      "type": "Catalog",
      "items": [
        {
          "id": "https://example.org/cat/climate-indices",
          "name": "Climate indices",
          "type": "Catalog",
          "items": [
            {
              "id": "https://example.org/dataset/temperature-anomaly",
              "name": "Global temperature anomaly dataset",
              "type": "Dataset"
            }
          ]
        }
      ]
    }
  ]
}

```

#### jsonld
```jsonld
{
  "@context": "https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/hierarchy/ogc/context.jsonld",
  "id": "https://example.org/cat/top",
  "name": "Earth observation definitions",
  "type": "TopCatalog",
  "items": [
    {
      "id": "https://example.org/cat/climate",
      "name": "Climate",
      "type": "Catalog",
      "items": [
        {
          "id": "https://example.org/cat/climate-indices",
          "name": "Climate indices",
          "type": "Catalog",
          "items": [
            {
              "id": "https://example.org/dataset/temperature-anomaly",
              "name": "Global temperature anomaly dataset",
              "type": "Dataset"
            }
          ]
        }
      ]
    }
  ]
}
```

#### ttl
```ttl
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .

<https://example.org/cat/top> a dcat:TopCatalog ;
    dcterms:hasPart <https://example.org/cat/climate> ;
    skos:prefLabel "Earth observation definitions" .

<https://example.org/cat/climate> a dcat:Catalog ;
    dcterms:hasPart <https://example.org/cat/climate-indices> ;
    skos:prefLabel "Climate" .

<https://example.org/cat/climate-indices> a dcat:Catalog ;
    dcterms:hasPart <https://example.org/dataset/temperature-anomaly> ;
    skos:prefLabel "Climate indices" .

<https://example.org/dataset/temperature-anomaly> a dcat:Dataset ;
    skos:prefLabel "Global temperature anomaly dataset" .


```


### TopCatalog with an Ontology and a Collection
A TopCatalog combining an Ontology entry with a Collection grouping a subset
of concepts.

#### json
```json
{
  "id": "https://example.org/cat/top",
  "name": "Vocabulary definitions",
  "type": "TopCatalog",
  "items": [
    {
      "id": "https://example.org/ont/units",
      "name": "Units of measure ontology",
      "type": "Ontology"
    },
    {
      "id": "https://example.org/col/lengths",
      "name": "Length-related concepts",
      "type": "Collection",
      "members": [
        { "id": "https://example.org/concept/metre", "name": "Metre", "type": "Concept" },
        { "id": "https://example.org/concept/foot", "name": "Foot", "type": "Concept" }
      ]
    }
  ]
}

```

#### jsonld
```jsonld
{
  "@context": "https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/hierarchy/ogc/context.jsonld",
  "id": "https://example.org/cat/top",
  "name": "Vocabulary definitions",
  "type": "TopCatalog",
  "items": [
    {
      "id": "https://example.org/ont/units",
      "name": "Units of measure ontology",
      "type": "Ontology"
    },
    {
      "id": "https://example.org/col/lengths",
      "name": "Length-related concepts",
      "type": "Collection",
      "members": [
        {
          "id": "https://example.org/concept/metre",
          "name": "Metre",
          "type": "Concept"
        },
        {
          "id": "https://example.org/concept/foot",
          "name": "Foot",
          "type": "Concept"
        }
      ]
    }
  ]
}
```

#### ttl
```ttl
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .

<https://example.org/cat/top> a dcat:TopCatalog ;
    dcterms:hasPart <https://example.org/col/lengths>,
        <https://example.org/ont/units> ;
    skos:prefLabel "Vocabulary definitions" .

<https://example.org/col/lengths> a skos:Collection ;
    skos:member <https://example.org/concept/foot>,
        <https://example.org/concept/metre> ;
    skos:prefLabel "Length-related concepts" .

<https://example.org/concept/foot> a skos:Concept ;
    skos:prefLabel "Foot" .

<https://example.org/concept/metre> a skos:Concept ;
    skos:prefLabel "Metre" .

<https://example.org/ont/units> a owl:Ontology ;
    skos:prefLabel "Units of measure ontology" .


```


### Concept with an implicit narrower hierarchy
A single root concept declares one nested concept under `narrower`, with no `broader` and
no `topConceptOf` authored anywhere (`type` is still required on every entry in this
hierarchy, unlike the default one). Semantic uplift entails `skos:broader` on the child
from the parent's `narrower`, anchors the child to the scheme via `skos:inScheme`
propagation, and marks only the root (which has no `skos:broader`) as `skos:topConceptOf`
the scheme.

#### json
```json
{
  "id": "https://example.org/cat/top",
  "name": "Simple definitions",
  "type": "TopCatalog",
  "items": [
    {
      "id": "https://example.org/simple-scheme",
      "name": "Simple scheme",
      "type": "ConceptScheme",
      "concepts": [
        {
          "id": "https://example.org/concept/root",
          "name": "Root concept",
          "type": "Concept",
          "narrower": [
            { "id": "https://example.org/concept/child", "name": "Child concept", "type": "Concept" }
          ]
        }
      ]
    }
  ]
}

```

#### jsonld
```jsonld
{
  "@context": "https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/hierarchy/ogc/context.jsonld",
  "id": "https://example.org/cat/top",
  "name": "Simple definitions",
  "type": "TopCatalog",
  "items": [
    {
      "id": "https://example.org/simple-scheme",
      "name": "Simple scheme",
      "type": "ConceptScheme",
      "concepts": [
        {
          "id": "https://example.org/concept/root",
          "name": "Root concept",
          "type": "Concept",
          "narrower": [
            {
              "id": "https://example.org/concept/child",
              "name": "Child concept",
              "type": "Concept"
            }
          ]
        }
      ]
    }
  ]
}
```

#### ttl
```ttl
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .

<https://example.org/cat/top> a dcat:TopCatalog ;
    dcterms:hasPart <https://example.org/simple-scheme> ;
    skos:prefLabel "Simple definitions" .

<https://example.org/concept/child> a skos:Concept ;
    skos:broader <https://example.org/concept/root> ;
    skos:inScheme <https://example.org/simple-scheme> ;
    skos:prefLabel "Child concept" .

<https://example.org/concept/root> a skos:Concept ;
    skos:inScheme <https://example.org/simple-scheme> ;
    skos:narrower <https://example.org/concept/child> ;
    skos:prefLabel "Root concept" ;
    skos:topConceptOf <https://example.org/simple-scheme> .

<https://example.org/simple-scheme> a skos:ConceptScheme ;
    skos:hasTopConcept <https://example.org/concept/root> ;
    skos:prefLabel "Simple scheme" .


```


### Concept scheme with a broader/narrower hierarchy
A concept scheme demonstrating `skos:broader` / `skos:narrower` relations: "Animal" has
"Mammal" and "Bird" nested inline as narrower concepts, "Mammal" in turn nests "Dog" and
"Cat". A separate top concept, "Pet", references "Dog" and "Cat" by id to add a second
`skos:broader` parent to each — a poly-hierarchy. Only "Animal" and "Pet" (which have no
`skos:broader`) end up as `skos:topConceptOf` the scheme.

#### json
```json
{
  "id": "https://example.org/cat/top",
  "name": "Taxonomy definitions",
  "type": "TopCatalog",
  "items": [
    {
      "id": "https://example.org/animal-scheme",
      "name": "Animal taxonomy",
      "type": "ConceptScheme",
      "concepts": [
        {
          "id": "https://example.org/concept/animal",
          "name": "Animal",
          "type": "Concept",
          "narrower": [
            {
              "id": "https://example.org/concept/mammal",
              "name": "Mammal",
              "type": "Concept",
              "narrower": [
                { "id": "https://example.org/concept/dog", "name": "Dog", "type": "Concept" },
                { "id": "https://example.org/concept/cat", "name": "Cat", "type": "Concept" }
              ]
            },
            {
              "id": "https://example.org/concept/bird",
              "name": "Bird",
              "type": "Concept"
            }
          ]
        },
        {
          "id": "https://example.org/concept/pet",
          "name": "Pet",
          "type": "Concept",
          "narrower": [
            "https://example.org/concept/dog",
            "https://example.org/concept/cat"
          ]
        }
      ]
    }
  ]
}

```

#### jsonld
```jsonld
{
  "@context": "https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/hierarchy/ogc/context.jsonld",
  "id": "https://example.org/cat/top",
  "name": "Taxonomy definitions",
  "type": "TopCatalog",
  "items": [
    {
      "id": "https://example.org/animal-scheme",
      "name": "Animal taxonomy",
      "type": "ConceptScheme",
      "concepts": [
        {
          "id": "https://example.org/concept/animal",
          "name": "Animal",
          "type": "Concept",
          "narrower": [
            {
              "id": "https://example.org/concept/mammal",
              "name": "Mammal",
              "type": "Concept",
              "narrower": [
                {
                  "id": "https://example.org/concept/dog",
                  "name": "Dog",
                  "type": "Concept"
                },
                {
                  "id": "https://example.org/concept/cat",
                  "name": "Cat",
                  "type": "Concept"
                }
              ]
            },
            {
              "id": "https://example.org/concept/bird",
              "name": "Bird",
              "type": "Concept"
            }
          ]
        },
        {
          "id": "https://example.org/concept/pet",
          "name": "Pet",
          "type": "Concept",
          "narrower": [
            "https://example.org/concept/dog",
            "https://example.org/concept/cat"
          ]
        }
      ]
    }
  ]
}
```

#### ttl
```ttl
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .

<https://example.org/cat/top> a dcat:TopCatalog ;
    dcterms:hasPart <https://example.org/animal-scheme> ;
    skos:prefLabel "Taxonomy definitions" .

<https://example.org/concept/bird> a skos:Concept ;
    skos:broader <https://example.org/concept/animal> ;
    skos:inScheme <https://example.org/animal-scheme> ;
    skos:prefLabel "Bird" .

<https://example.org/concept/cat> a skos:Concept ;
    skos:broader <https://example.org/concept/mammal>,
        <https://example.org/concept/pet> ;
    skos:inScheme <https://example.org/animal-scheme> ;
    skos:prefLabel "Cat" .

<https://example.org/concept/dog> a skos:Concept ;
    skos:broader <https://example.org/concept/mammal>,
        <https://example.org/concept/pet> ;
    skos:inScheme <https://example.org/animal-scheme> ;
    skos:prefLabel "Dog" .

<https://example.org/concept/animal> a skos:Concept ;
    skos:inScheme <https://example.org/animal-scheme> ;
    skos:narrower <https://example.org/concept/bird>,
        <https://example.org/concept/mammal> ;
    skos:prefLabel "Animal" ;
    skos:topConceptOf <https://example.org/animal-scheme> .

<https://example.org/concept/mammal> a skos:Concept ;
    skos:broader <https://example.org/concept/animal> ;
    skos:inScheme <https://example.org/animal-scheme> ;
    skos:narrower <https://example.org/concept/cat>,
        <https://example.org/concept/dog> ;
    skos:prefLabel "Mammal" .

<https://example.org/concept/pet> a skos:Concept ;
    skos:inScheme <https://example.org/animal-scheme> ;
    skos:narrower <https://example.org/concept/cat>,
        <https://example.org/concept/dog> ;
    skos:prefLabel "Pet" ;
    skos:topConceptOf <https://example.org/animal-scheme> .

<https://example.org/animal-scheme> a skos:ConceptScheme ;
    skos:hasTopConcept <https://example.org/concept/animal>,
        <https://example.org/concept/pet> ;
    skos:prefLabel "Animal taxonomy" .


```


### TopCatalog with externally-defined items
A standalone TopCatalog whose items are plain IRI strings rather than embedded objects,
for resources that are already defined elsewhere — possibly in a different graph. Each
string becomes a `dct:hasPart` link to that IRI, without asserting any type or label
locally.

#### json
```json
{
  "id": "https://example.org/cat/data-models",
  "name": "OGC data models (partial)",
  "type": "TopCatalog",
  "items": [
    "http://www.opengis.net/def/observationType/timeseriesML",
    "http://www.opengis.net/def/waterml",
    "http://www.opengis.net/ont/geosparql"
  ]
}

```

#### jsonld
```jsonld
{
  "@context": "https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/hierarchy/ogc/context.jsonld",
  "id": "https://example.org/cat/data-models",
  "name": "OGC data models (partial)",
  "type": "TopCatalog",
  "items": [
    "http://www.opengis.net/def/observationType/timeseriesML",
    "http://www.opengis.net/def/waterml",
    "http://www.opengis.net/ont/geosparql"
  ]
}
```

#### ttl
```ttl
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .

<https://example.org/cat/data-models> a dcat:TopCatalog ;
    dcterms:hasPart <http://www.opengis.net/def/observationType/timeseriesML>,
        <http://www.opengis.net/def/waterml>,
        <http://www.opengis.net/ont/geosparql> ;
    skos:prefLabel "OGC data models (partial)" .


```


### Multiple TopCatalogs with namespace prefix declarations
An array mixing namespace prefix declarations with two independent TopCatalogs.
Prefix declarations have no `id` and are uplifted as standalone blank nodes,
unrelated to the catalogs or to each other — they are not part of the hierarchy,
just global CURIE hints for Prez.

#### json
```json
[
  { "prefix": "ex", "uri": "https://example.org/" },
  {
    "id": "https://example.org/cat/standards",
    "name": "OGC Standards",
    "type": "TopCatalog",
    "items": [
      {
        "id": "https://example.org/resource/abstract-spec",
        "name": "Abstract Specification",
        "type": "Resource"
      }
    ]
  },
  {
    "id": "https://example.org/cat/registers",
    "name": "OGC Registers",
    "type": "TopCatalog",
    "items": [
      {
        "id": "https://example.org/resource/codelist",
        "name": "Codelist register",
        "type": "Resource"
      }
    ]
  }
]

```

#### jsonld
```jsonld
{
  "@context": "https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/hierarchy/ogc/context.jsonld",
  "@graph": [
    {
      "prefix": "ex",
      "uri": "https://example.org/"
    },
    {
      "id": "https://example.org/cat/standards",
      "name": "OGC Standards",
      "type": "TopCatalog",
      "items": [
        {
          "id": "https://example.org/resource/abstract-spec",
          "name": "Abstract Specification",
          "type": "Resource"
        }
      ]
    },
    {
      "id": "https://example.org/cat/registers",
      "name": "OGC Registers",
      "type": "TopCatalog",
      "items": [
        {
          "id": "https://example.org/resource/codelist",
          "name": "Codelist register",
          "type": "Resource"
        }
      ]
    }
  ]
}
```

#### ttl
```ttl
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .
@prefix vann: <http://purl.org/vocab/vann/> .

<https://example.org/cat/registers> a dcat:TopCatalog ;
    dcterms:hasPart <https://example.org/resource/codelist> ;
    skos:prefLabel "OGC Registers" .

<https://example.org/cat/standards> a dcat:TopCatalog ;
    dcterms:hasPart <https://example.org/resource/abstract-spec> ;
    skos:prefLabel "OGC Standards" .

<https://example.org/resource/abstract-spec> a dcat:Resource ;
    skos:prefLabel "Abstract Specification" .

<https://example.org/resource/codelist> a dcat:Resource ;
    skos:prefLabel "Codelist register" .

[] vann:preferredNamespacePrefix "ex" ;
    vann:preferredNamespaceUri "https://example.org/" .


```

## Schema

```yaml
$defs:
  Text:
    description: 'Either a plain string or a languageCode -> value mapping for multi-language
      strings.

      '
    oneOf:
    - type: string
    - type: object
      propertyNames:
        pattern: ^[a-zA-Z][a-zA-Z0-9]*(-[a-zA-Z0-9]+)*$
      additionalProperties:
        type: string
  NamedResource:
    type: object
    required:
    - id
    - name
    - type
    properties:
      id:
        type: string
        x-jsonld-id: '@id'
      name:
        $ref: '#/$defs/Text'
        x-jsonld-id: http://www.w3.org/2004/02/skos/core#prefLabel
        x-jsonld-container: '@language'
      definition:
        $ref: '#/$defs/Text'
        x-jsonld-id: http://www.w3.org/2004/02/skos/core#definition
        x-jsonld-container: '@language'
  TopCatalog:
    allOf:
    - $ref: '#/$defs/NamedResource'
    properties:
      type:
        const: TopCatalog
        x-jsonld-id: '@type'
      items:
        type: array
        items:
          oneOf:
          - type: string
          - $ref: '#/$defs/CatalogEntry'
        x-jsonld-id: http://purl.org/dc/terms/hasPart
        x-jsonld-type: '@id'
  Catalog:
    allOf:
    - $ref: '#/$defs/NamedResource'
    properties:
      type:
        const: Catalog
        x-jsonld-id: '@type'
      items:
        type: array
        items:
          oneOf:
          - type: string
          - $ref: '#/$defs/CatalogEntry'
        x-jsonld-id: http://purl.org/dc/terms/hasPart
        x-jsonld-type: '@id'
  ConceptScheme:
    allOf:
    - $ref: '#/$defs/NamedResource'
    properties:
      type:
        const: ConceptScheme
        x-jsonld-id: '@type'
      concepts:
        type: array
        items:
          $ref: '#/$defs/Concept'
        x-jsonld-reverse: skos:inScheme
  Collection:
    allOf:
    - $ref: '#/$defs/NamedResource'
    properties:
      type:
        const: Collection
        x-jsonld-id: '@type'
      members:
        type: array
        items:
          $ref: '#/$defs/Concept'
        x-jsonld-id: http://www.w3.org/2004/02/skos/core#member
        x-jsonld-type: '@id'
  Concept:
    allOf:
    - $ref: '#/$defs/NamedResource'
    properties:
      type:
        const: Concept
        x-jsonld-id: '@type'
      broader:
        type: array
        items:
          type: string
        x-jsonld-id: http://www.w3.org/2004/02/skos/core#broader
        x-jsonld-type: '@id'
      narrower:
        type: array
        items:
          oneOf:
          - type: string
          - $ref: '#/$defs/Concept'
        x-jsonld-id: http://www.w3.org/2004/02/skos/core#narrower
        x-jsonld-type: '@id'
  CatalogEntry:
    allOf:
    - $ref: '#/$defs/NamedResource'
    oneOf:
    - $ref: '#/$defs/Catalog'
    - $ref: '#/$defs/ConceptScheme'
    - $ref: '#/$defs/Collection'
    - properties:
        type:
          enum:
          - Dataset
          - Resource
          - Ontology
          x-jsonld-id: '@type'
anyOf:
- $ref: '#/$defs/TopCatalog'
- type: array
  items:
    oneOf:
    - $ref: '#/$defs/TopCatalog'
    - $ref: https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/prefix/schema.yaml
x-jsonld-extra-terms:
  id: '@id'
  name:
    x-jsonld-id: http://www.w3.org/2004/02/skos/core#prefLabel
    x-jsonld-container: '@language'
  definition:
    x-jsonld-id: http://www.w3.org/2004/02/skos/core#definition
    x-jsonld-container: '@language'
  TopCatalog: http://www.w3.org/ns/dcat#TopCatalog
  Catalog: http://www.w3.org/ns/dcat#Catalog
  ConceptScheme: http://www.w3.org/2004/02/skos/core#ConceptScheme
  Collection: http://www.w3.org/2004/02/skos/core#Collection
  Concept: http://www.w3.org/2004/02/skos/core#Concept
  Dataset: http://www.w3.org/ns/dcat#Dataset
  Resource: http://www.w3.org/ns/dcat#Resource
  Ontology: http://www.w3.org/2002/07/owl#Ontology
  members:
    x-jsonld-id: http://www.w3.org/2004/02/skos/core#member
    x-jsonld-type: '@id'
  broader:
    x-jsonld-id: http://www.w3.org/2004/02/skos/core#broader
    x-jsonld-type: '@id'
  narrower:
    x-jsonld-id: http://www.w3.org/2004/02/skos/core#narrower
    x-jsonld-type: '@id'
  concepts:
    x-jsonld-reverse: skos:inScheme
x-jsonld-prefixes:
  dct: http://purl.org/dc/terms/
  skos: http://www.w3.org/2004/02/skos/core#
  dcat: http://www.w3.org/ns/dcat#
  owl: http://www.w3.org/2002/07/owl#

```

Links to the schema:

* YAML version: [schema.yaml](https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/hierarchy/ogc/schema.json)
* JSON version: [schema.json](https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/hierarchy/ogc/schema.yaml)


# JSON-LD Context

```jsonld
{
  "@context": {
    "id": "@id",
    "name": {
      "@id": "skos:prefLabel",
      "@container": "@language"
    },
    "definition": {
      "@id": "skos:definition",
      "@container": "@language"
    },
    "type": "@type",
    "items": {
      "@id": "dct:hasPart",
      "@type": "@id"
    },
    "prefix": "vann:preferredNamespacePrefix",
    "uri": "vann:preferredNamespaceUri",
    "TopCatalog": "dcat:TopCatalog",
    "Catalog": "dcat:Catalog",
    "ConceptScheme": "skos:ConceptScheme",
    "Collection": "skos:Collection",
    "Concept": "skos:Concept",
    "Dataset": "dcat:Dataset",
    "Resource": "dcat:Resource",
    "Ontology": "owl:Ontology",
    "members": {
      "@id": "skos:member",
      "@type": "@id"
    },
    "broader": {
      "@id": "skos:broader",
      "@type": "@id"
    },
    "narrower": {
      "@id": "skos:narrower",
      "@type": "@id"
    },
    "concepts": {
      "@reverse": "skos:inScheme"
    },
    "dct": "http://purl.org/dc/terms/",
    "skos": "http://www.w3.org/2004/02/skos/core#",
    "dcat": "http://www.w3.org/ns/dcat#",
    "owl": "http://www.w3.org/2002/07/owl#",
    "vann": "http://purl.org/vocab/vann/",
    "@version": 1.1
  }
}
```

You can find the full JSON-LD context here:
[context.jsonld](https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/hierarchy/ogc/context.jsonld)


# For developers

The source code for this Building Block can be found in the following repository:

* URL: [https://github.com/ogcincubator/bblocks-prez](https://github.com/ogcincubator/bblocks-prez)
* Path: `_sources/hierarchy/ogc`

