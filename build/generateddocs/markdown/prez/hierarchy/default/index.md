
# Prez default hierarchy (Schema)

`ogc.prez.hierarchy.default` *v0.1*

Helps create a default Prez hierarchy

[*Status*](http://www.opengis.net/def/status): Under development

## Description

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

## Examples

### Minimal catalog
A single catalog containing one concept scheme with two concepts.
Type annotations are omitted; they will be inferred during semantic uplift.

#### json
```json
{
  "id": "https://example.org/my-catalog",
  "name": "My Catalog",
  "items": [
    {
      "id": "https://example.org/my-scheme",
      "name": "My Concept Scheme",
      "concepts": [
        {
          "id": "https://example.org/concept-a",
          "name": "Concept A"
        },
        {
          "id": "https://example.org/concept-b",
          "name": "Concept B"
        }
      ]
    }
  ]
}

```

#### jsonld
```jsonld
{
  "@context": [
    {
      "ex": "https://example.org/"
    },
    "https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/hierarchy/default/context.jsonld"
  ],
  "id": "https://example.org/my-catalog",
  "name": "My Catalog",
  "items": [
    {
      "id": "https://example.org/my-scheme",
      "name": "My Concept Scheme",
      "concepts": [
        {
          "id": "https://example.org/concept-a",
          "name": "Concept A",
          "type": "Concept"
        },
        {
          "id": "https://example.org/concept-b",
          "name": "Concept B",
          "type": "Concept"
        }
      ],
      "type": "ConceptScheme"
    }
  ],
  "type": "Catalog"
}
```

#### ttl
```ttl
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix ex: <https://example.org/> .
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .

ex:my-catalog a dcat:Catalog ;
    dcterms:hasPart ex:my-scheme ;
    skos:prefLabel "My Catalog" .

ex:concept-a a skos:Concept ;
    skos:inScheme ex:my-scheme ;
    skos:prefLabel "Concept A" ;
    skos:topConceptOf ex:my-scheme .

ex:concept-b a skos:Concept ;
    skos:inScheme ex:my-scheme ;
    skos:prefLabel "Concept B" ;
    skos:topConceptOf ex:my-scheme .

ex:my-scheme a skos:ConceptScheme ;
    skos:hasTopConcept ex:concept-a,
        ex:concept-b ;
    skos:prefLabel "My Concept Scheme" .


```


### Minimal catalog with base URL
Same minimal structure as above, but wrapped in the envelope form that provides
a `base` URL. IDs are relative to that base, keeping them short.

#### json
```json
{
  "base": "https://example.org/",
  "catalogs": [
    {
      "id": "my-catalog",
      "name": "My Catalog",
      "items": [
        {
          "id": "my-scheme",
          "name": "My Concept Scheme",
          "concepts": [
            {
              "id": "concept-a",
              "name": "Concept A"
            },
            {
              "id": "concept-b",
              "name": "Concept B"
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
  "@context": [
    {
      "ex": "https://example.org/"
    },
    "https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/hierarchy/default/context.jsonld"
  ],
  "base": "https://example.org/",
  "catalogs": [
    {
      "id": "my-catalog",
      "name": "My Catalog",
      "items": [
        {
          "id": "my-scheme",
          "name": "My Concept Scheme",
          "concepts": [
            {
              "id": "concept-a",
              "name": "Concept A",
              "type": "Concept"
            },
            {
              "id": "concept-b",
              "name": "Concept B",
              "type": "Concept"
            }
          ],
          "type": "ConceptScheme"
        }
      ],
      "type": "Catalog"
    }
  ]
}
```

#### ttl
```ttl
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix ex: <https://example.org/> .
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .

ex:my-catalog a dcat:Catalog ;
    dcterms:hasPart ex:my-scheme ;
    skos:prefLabel "My Catalog" .

ex:concept-a a skos:Concept ;
    skos:inScheme ex:my-scheme ;
    skos:prefLabel "Concept A" ;
    skos:topConceptOf ex:my-scheme .

ex:concept-b a skos:Concept ;
    skos:inScheme ex:my-scheme ;
    skos:prefLabel "Concept B" ;
    skos:topConceptOf ex:my-scheme .

ex:my-scheme a skos:ConceptScheme ;
    skos:hasTopConcept ex:concept-a,
        ex:concept-b ;
    skos:prefLabel "My Concept Scheme" .


```


### Catalog with concept scheme and collection
A catalog containing a concept scheme (all animal species) and a collection
(domestic pets) referencing a subset of those concepts.

#### json
```json
{
  "id": "https://example.org/animals",
  "name": "Animal Taxonomy",
  "items": [
    {
      "id": "https://example.org/animals/scheme",
      "name": "Animal Species",
      "definition": "A classification of common animal species.",
      "concepts": [
        {
          "id": "https://example.org/animals/dog",
          "name": "Dog"
        },
        {
          "id": "https://example.org/animals/cat",
          "name": "Cat"
        },
        {
          "id": "https://example.org/animals/eagle",
          "name": "Eagle"
        }
      ]
    },
    {
      "id": "https://example.org/animals/pets",
      "name": "Domestic Pets",
      "type": "Collection",
      "members": [
        {
          "id": "https://example.org/animals/dog",
          "name": "Dog"
        },
        {
          "id": "https://example.org/animals/cat",
          "name": "Cat"
        }
      ]
    }
  ]
}

```

#### jsonld
```jsonld
{
  "@context": [
    {
      "ex": "https://example.org/"
    },
    "https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/hierarchy/default/context.jsonld"
  ],
  "id": "https://example.org/animals",
  "name": "Animal Taxonomy",
  "items": [
    {
      "id": "https://example.org/animals/scheme",
      "name": "Animal Species",
      "definition": "A classification of common animal species.",
      "concepts": [
        {
          "id": "https://example.org/animals/dog",
          "name": "Dog",
          "type": "Concept"
        },
        {
          "id": "https://example.org/animals/cat",
          "name": "Cat",
          "type": "Concept"
        },
        {
          "id": "https://example.org/animals/eagle",
          "name": "Eagle",
          "type": "Concept"
        }
      ],
      "type": "ConceptScheme"
    },
    {
      "id": "https://example.org/animals/pets",
      "name": "Domestic Pets",
      "type": "Collection",
      "members": [
        {
          "id": "https://example.org/animals/dog",
          "name": "Dog",
          "type": "Concept"
        },
        {
          "id": "https://example.org/animals/cat",
          "name": "Cat",
          "type": "Concept"
        }
      ]
    }
  ],
  "type": "Catalog"
}
```

#### ttl
```ttl
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix ex: <https://example.org/> .
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .

ex:animals a dcat:Catalog ;
    dcterms:hasPart <https://example.org/animals/pets>,
        <https://example.org/animals/scheme> ;
    skos:prefLabel "Animal Taxonomy" .

<https://example.org/animals/eagle> a skos:Concept ;
    skos:inScheme <https://example.org/animals/scheme> ;
    skos:prefLabel "Eagle" ;
    skos:topConceptOf <https://example.org/animals/scheme> .

<https://example.org/animals/pets> a skos:Collection ;
    skos:member <https://example.org/animals/cat>,
        <https://example.org/animals/dog> ;
    skos:prefLabel "Domestic Pets" .

<https://example.org/animals/cat> a skos:Concept ;
    skos:inScheme <https://example.org/animals/scheme> ;
    skos:prefLabel "Cat" ;
    skos:topConceptOf <https://example.org/animals/scheme> .

<https://example.org/animals/dog> a skos:Concept ;
    skos:inScheme <https://example.org/animals/scheme> ;
    skos:prefLabel "Dog" ;
    skos:topConceptOf <https://example.org/animals/scheme> .

<https://example.org/animals/scheme> a skos:ConceptScheme ;
    skos:definition "A classification of common animal species." ;
    skos:hasTopConcept <https://example.org/animals/cat>,
        <https://example.org/animals/dog>,
        <https://example.org/animals/eagle> ;
    skos:prefLabel "Animal Species" .


```


### Catalog mixing concept scheme, collection, dataset, and resource
A land cover catalog combining a concept scheme (LCCS-based types), a collection
(urban subset), a dataset (global map), and an external resource (FAO specification).

#### json
```json
{
  "id": "https://example.org/land-cover",
  "name": "Land Cover Catalog",
  "items": [
    {
      "id": "https://example.org/land-cover/types",
      "name": "Land Cover Types",
      "definition": "LCCS-based classification of land cover categories.",
      "concepts": [
        {
          "id": "https://example.org/land-cover/types/cropland",
          "name": "Cropland"
        },
        {
          "id": "https://example.org/land-cover/types/forest",
          "name": "Forest"
        },
        {
          "id": "https://example.org/land-cover/types/urban",
          "name": "Urban"
        },
        {
          "id": "https://example.org/land-cover/types/water",
          "name": "Water Bodies"
        }
      ]
    },
    {
      "id": "https://example.org/land-cover/urban-types",
      "name": "Urban Land Cover",
      "type": "Collection",
      "definition": "Subset of land cover types applicable to urban areas.",
      "members": [
        {
          "id": "https://example.org/land-cover/types/urban",
          "name": "Urban"
        }
      ]
    },
    {
      "id": "https://example.org/land-cover/global-2020",
      "name": "Global Land Cover Map 2020",
      "type": "Dataset",
      "definition": "30m resolution global land cover product derived from Sentinel-2 imagery."
    },
    {
      "id": "https://example.org/land-cover/lccs-spec",
      "name": "FAO LCCS Specification",
      "type": "Resource",
      "definition": "Land Cover Classification System specification published by FAO."
    }
  ]
}

```

#### jsonld
```jsonld
{
  "@context": [
    {
      "ex": "https://example.org/"
    },
    "https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/hierarchy/default/context.jsonld"
  ],
  "id": "https://example.org/land-cover",
  "name": "Land Cover Catalog",
  "items": [
    {
      "id": "https://example.org/land-cover/types",
      "name": "Land Cover Types",
      "definition": "LCCS-based classification of land cover categories.",
      "concepts": [
        {
          "id": "https://example.org/land-cover/types/cropland",
          "name": "Cropland",
          "type": "Concept"
        },
        {
          "id": "https://example.org/land-cover/types/forest",
          "name": "Forest",
          "type": "Concept"
        },
        {
          "id": "https://example.org/land-cover/types/urban",
          "name": "Urban",
          "type": "Concept"
        },
        {
          "id": "https://example.org/land-cover/types/water",
          "name": "Water Bodies",
          "type": "Concept"
        }
      ],
      "type": "ConceptScheme"
    },
    {
      "id": "https://example.org/land-cover/urban-types",
      "name": "Urban Land Cover",
      "type": "Collection",
      "definition": "Subset of land cover types applicable to urban areas.",
      "members": [
        {
          "id": "https://example.org/land-cover/types/urban",
          "name": "Urban",
          "type": "Concept"
        }
      ]
    },
    {
      "id": "https://example.org/land-cover/global-2020",
      "name": "Global Land Cover Map 2020",
      "type": "Dataset",
      "definition": "30m resolution global land cover product derived from Sentinel-2 imagery."
    },
    {
      "id": "https://example.org/land-cover/lccs-spec",
      "name": "FAO LCCS Specification",
      "type": "Resource",
      "definition": "Land Cover Classification System specification published by FAO."
    }
  ],
  "type": "Catalog"
}
```

#### ttl
```ttl
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix ex: <https://example.org/> .
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .

ex:land-cover a dcat:Catalog ;
    dcterms:hasPart <https://example.org/land-cover/global-2020>,
        <https://example.org/land-cover/lccs-spec>,
        <https://example.org/land-cover/types>,
        <https://example.org/land-cover/urban-types> ;
    skos:prefLabel "Land Cover Catalog" .

<https://example.org/land-cover/global-2020> a dcat:Dataset ;
    skos:definition "30m resolution global land cover product derived from Sentinel-2 imagery." ;
    skos:prefLabel "Global Land Cover Map 2020" .

<https://example.org/land-cover/lccs-spec> a dcat:Resource ;
    skos:definition "Land Cover Classification System specification published by FAO." ;
    skos:prefLabel "FAO LCCS Specification" .

<https://example.org/land-cover/types/cropland> a skos:Concept ;
    skos:inScheme <https://example.org/land-cover/types> ;
    skos:prefLabel "Cropland" ;
    skos:topConceptOf <https://example.org/land-cover/types> .

<https://example.org/land-cover/types/forest> a skos:Concept ;
    skos:inScheme <https://example.org/land-cover/types> ;
    skos:prefLabel "Forest" ;
    skos:topConceptOf <https://example.org/land-cover/types> .

<https://example.org/land-cover/types/water> a skos:Concept ;
    skos:inScheme <https://example.org/land-cover/types> ;
    skos:prefLabel "Water Bodies" ;
    skos:topConceptOf <https://example.org/land-cover/types> .

<https://example.org/land-cover/urban-types> a skos:Collection ;
    skos:definition "Subset of land cover types applicable to urban areas." ;
    skos:member <https://example.org/land-cover/types/urban> ;
    skos:prefLabel "Urban Land Cover" .

<https://example.org/land-cover/types/urban> a skos:Concept ;
    skos:inScheme <https://example.org/land-cover/types> ;
    skos:prefLabel "Urban" ;
    skos:topConceptOf <https://example.org/land-cover/types> .

<https://example.org/land-cover/types> a skos:ConceptScheme ;
    skos:definition "LCCS-based classification of land cover categories." ;
    skos:hasTopConcept <https://example.org/land-cover/types/cropland>,
        <https://example.org/land-cover/types/forest>,
        <https://example.org/land-cover/types/urban>,
        <https://example.org/land-cover/types/water> ;
    skos:prefLabel "Land Cover Types" .


```


### Multiple mixed catalogs
Two domain catalogs (climate and hydrology), each containing concept schemes,
collections, datasets, and resources. Demonstrates how different entry types
coexist within and across catalogs.

#### json
```json
[
  {
    "id": "https://example.org/climate",
    "name": {"en": "Climate Observations", "fr": "Observations Climatiques"},
    "items": [
      {
        "id": "https://example.org/climate/variables",
        "name": "Climate Variables",
        "definition": "Essential climate variables as defined by GCOS.",
        "concepts": [
          {
            "id": "https://example.org/climate/variables/temperature",
            "name": "Air Temperature"
          },
          {
            "id": "https://example.org/climate/variables/precipitation",
            "name": "Precipitation"
          },
          {
            "id": "https://example.org/climate/variables/humidity",
            "name": "Relative Humidity"
          }
        ]
      },
      {
        "id": "https://example.org/climate/surface-vars",
        "name": "Surface Variables",
        "type": "Collection",
        "definition": "Climate variables measured at surface level.",
        "members": [
          {
            "id": "https://example.org/climate/variables/temperature",
            "name": "Air Temperature"
          },
          {
            "id": "https://example.org/climate/variables/precipitation",
            "name": "Precipitation"
          }
        ]
      },
      {
        "id": "https://example.org/climate/era5",
        "name": "ERA5 Reanalysis",
        "type": "Dataset",
        "definition": "ECMWF global climate reanalysis dataset from 1940 to present."
      },
      {
        "id": "https://example.org/climate/gcos-spec",
        "name": "GCOS Essential Climate Variables",
        "type": "Resource",
        "definition": "Official GCOS specification for essential climate variables."
      }
    ]
  },
  {
    "id": "https://example.org/hydrology",
    "name": "Hydrological Data",
    "items": [
      {
        "id": "https://example.org/hydrology/features",
        "name": "Hydrological Features",
        "concepts": [
          {
            "id": "https://example.org/hydrology/features/river",
            "name": "River"
          },
          {
            "id": "https://example.org/hydrology/features/lake",
            "name": "Lake"
          },
          {
            "id": "https://example.org/hydrology/features/aquifer",
            "name": "Aquifer"
          }
        ]
      },
      {
        "id": "https://example.org/hydrology/surface-water",
        "name": "Surface Water Bodies",
        "type": "Collection",
        "members": [
          {
            "id": "https://example.org/hydrology/features/river",
            "name": "River"
          },
          {
            "id": "https://example.org/hydrology/features/lake",
            "name": "Lake"
          }
        ]
      },
      {
        "id": "https://example.org/hydrology/gsim",
        "name": "GSIM Streamflow Dataset",
        "type": "Dataset",
        "definition": "Global Streamflow Indices and Metadata archive."
      },
      {
        "id": "https://example.org/hydrology/hydrosheds",
        "name": "HydroSHEDS",
        "type": "Dataset",
        "definition": "Hydrographic data derived from SRTM elevation data at multiple scales."
      },
      {
        "id": "https://example.org/hydrology/wmo-guide",
        "name": "WMO Hydrological Practices Guide",
        "type": "Resource",
        "definition": "WMO reference guide for hydrological observation and data management."
      }
    ]
  }
]

```

#### jsonld
```jsonld
{
  "@context": [
    {
      "ex": "https://example.org/"
    },
    "https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/hierarchy/default/context.jsonld"
  ],
  "@graph": [
    {
      "id": "https://example.org/climate",
      "name": {
        "en": "Climate Observations",
        "fr": "Observations Climatiques"
      },
      "items": [
        {
          "id": "https://example.org/climate/variables",
          "name": "Climate Variables",
          "definition": "Essential climate variables as defined by GCOS.",
          "concepts": [
            {
              "id": "https://example.org/climate/variables/temperature",
              "name": "Air Temperature",
              "type": "Concept"
            },
            {
              "id": "https://example.org/climate/variables/precipitation",
              "name": "Precipitation",
              "type": "Concept"
            },
            {
              "id": "https://example.org/climate/variables/humidity",
              "name": "Relative Humidity",
              "type": "Concept"
            }
          ],
          "type": "ConceptScheme"
        },
        {
          "id": "https://example.org/climate/surface-vars",
          "name": "Surface Variables",
          "type": "Collection",
          "definition": "Climate variables measured at surface level.",
          "members": [
            {
              "id": "https://example.org/climate/variables/temperature",
              "name": "Air Temperature",
              "type": "Concept"
            },
            {
              "id": "https://example.org/climate/variables/precipitation",
              "name": "Precipitation",
              "type": "Concept"
            }
          ]
        },
        {
          "id": "https://example.org/climate/era5",
          "name": "ERA5 Reanalysis",
          "type": "Dataset",
          "definition": "ECMWF global climate reanalysis dataset from 1940 to present."
        },
        {
          "id": "https://example.org/climate/gcos-spec",
          "name": "GCOS Essential Climate Variables",
          "type": "Resource",
          "definition": "Official GCOS specification for essential climate variables."
        }
      ],
      "type": "Catalog"
    },
    {
      "id": "https://example.org/hydrology",
      "name": "Hydrological Data",
      "items": [
        {
          "id": "https://example.org/hydrology/features",
          "name": "Hydrological Features",
          "concepts": [
            {
              "id": "https://example.org/hydrology/features/river",
              "name": "River",
              "type": "Concept"
            },
            {
              "id": "https://example.org/hydrology/features/lake",
              "name": "Lake",
              "type": "Concept"
            },
            {
              "id": "https://example.org/hydrology/features/aquifer",
              "name": "Aquifer",
              "type": "Concept"
            }
          ],
          "type": "ConceptScheme"
        },
        {
          "id": "https://example.org/hydrology/surface-water",
          "name": "Surface Water Bodies",
          "type": "Collection",
          "members": [
            {
              "id": "https://example.org/hydrology/features/river",
              "name": "River",
              "type": "Concept"
            },
            {
              "id": "https://example.org/hydrology/features/lake",
              "name": "Lake",
              "type": "Concept"
            }
          ]
        },
        {
          "id": "https://example.org/hydrology/gsim",
          "name": "GSIM Streamflow Dataset",
          "type": "Dataset",
          "definition": "Global Streamflow Indices and Metadata archive."
        },
        {
          "id": "https://example.org/hydrology/hydrosheds",
          "name": "HydroSHEDS",
          "type": "Dataset",
          "definition": "Hydrographic data derived from SRTM elevation data at multiple scales."
        },
        {
          "id": "https://example.org/hydrology/wmo-guide",
          "name": "WMO Hydrological Practices Guide",
          "type": "Resource",
          "definition": "WMO reference guide for hydrological observation and data management."
        }
      ],
      "type": "Catalog"
    }
  ]
}
```

#### ttl
```ttl
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix ex: <https://example.org/> .
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .

ex:climate a dcat:Catalog ;
    dcterms:hasPart <https://example.org/climate/era5>,
        <https://example.org/climate/gcos-spec>,
        <https://example.org/climate/surface-vars>,
        <https://example.org/climate/variables> ;
    skos:prefLabel "Climate Observations"@en,
        "Observations Climatiques"@fr .

ex:hydrology a dcat:Catalog ;
    dcterms:hasPart <https://example.org/hydrology/features>,
        <https://example.org/hydrology/gsim>,
        <https://example.org/hydrology/hydrosheds>,
        <https://example.org/hydrology/surface-water>,
        <https://example.org/hydrology/wmo-guide> ;
    skos:prefLabel "Hydrological Data" .

<https://example.org/climate/era5> a dcat:Dataset ;
    skos:definition "ECMWF global climate reanalysis dataset from 1940 to present." ;
    skos:prefLabel "ERA5 Reanalysis" .

<https://example.org/climate/gcos-spec> a dcat:Resource ;
    skos:definition "Official GCOS specification for essential climate variables." ;
    skos:prefLabel "GCOS Essential Climate Variables" .

<https://example.org/climate/surface-vars> a skos:Collection ;
    skos:definition "Climate variables measured at surface level." ;
    skos:member <https://example.org/climate/variables/precipitation>,
        <https://example.org/climate/variables/temperature> ;
    skos:prefLabel "Surface Variables" .

<https://example.org/climate/variables/humidity> a skos:Concept ;
    skos:inScheme <https://example.org/climate/variables> ;
    skos:prefLabel "Relative Humidity" ;
    skos:topConceptOf <https://example.org/climate/variables> .

<https://example.org/hydrology/features/aquifer> a skos:Concept ;
    skos:inScheme <https://example.org/hydrology/features> ;
    skos:prefLabel "Aquifer" ;
    skos:topConceptOf <https://example.org/hydrology/features> .

<https://example.org/hydrology/gsim> a dcat:Dataset ;
    skos:definition "Global Streamflow Indices and Metadata archive." ;
    skos:prefLabel "GSIM Streamflow Dataset" .

<https://example.org/hydrology/hydrosheds> a dcat:Dataset ;
    skos:definition "Hydrographic data derived from SRTM elevation data at multiple scales." ;
    skos:prefLabel "HydroSHEDS" .

<https://example.org/hydrology/surface-water> a skos:Collection ;
    skos:member <https://example.org/hydrology/features/lake>,
        <https://example.org/hydrology/features/river> ;
    skos:prefLabel "Surface Water Bodies" .

<https://example.org/hydrology/wmo-guide> a dcat:Resource ;
    skos:definition "WMO reference guide for hydrological observation and data management." ;
    skos:prefLabel "WMO Hydrological Practices Guide" .

<https://example.org/climate/variables/precipitation> a skos:Concept ;
    skos:inScheme <https://example.org/climate/variables> ;
    skos:prefLabel "Precipitation" ;
    skos:topConceptOf <https://example.org/climate/variables> .

<https://example.org/climate/variables/temperature> a skos:Concept ;
    skos:inScheme <https://example.org/climate/variables> ;
    skos:prefLabel "Air Temperature" ;
    skos:topConceptOf <https://example.org/climate/variables> .

<https://example.org/hydrology/features/lake> a skos:Concept ;
    skos:inScheme <https://example.org/hydrology/features> ;
    skos:prefLabel "Lake" ;
    skos:topConceptOf <https://example.org/hydrology/features> .

<https://example.org/hydrology/features/river> a skos:Concept ;
    skos:inScheme <https://example.org/hydrology/features> ;
    skos:prefLabel "River" ;
    skos:topConceptOf <https://example.org/hydrology/features> .

<https://example.org/climate/variables> a skos:ConceptScheme ;
    skos:definition "Essential climate variables as defined by GCOS." ;
    skos:hasTopConcept <https://example.org/climate/variables/humidity>,
        <https://example.org/climate/variables/precipitation>,
        <https://example.org/climate/variables/temperature> ;
    skos:prefLabel "Climate Variables" .

<https://example.org/hydrology/features> a skos:ConceptScheme ;
    skos:hasTopConcept <https://example.org/hydrology/features/aquifer>,
        <https://example.org/hydrology/features/lake>,
        <https://example.org/hydrology/features/river> ;
    skos:prefLabel "Hydrological Features" .


```


### Multiple catalogs with multilingual labels
An array of two catalogs. The first covers environmental data and demonstrates
multilingual `name` values and a collection grouping a subset of a concept
scheme. The second is a simpler species registry.

#### json
```json
[
  {
    "id": "https://example.org/env",
    "name": {"en": "Environmental Data", "es": "Datos Ambientales"},
    "items": [
      {
        "id": "https://example.org/env/habitats",
        "name": {"en": "Habitat Types", "es": "Tipos de Hábitat"},
        "definition": "Classification of natural habitats.",
        "concepts": [
          {
            "id": "https://example.org/env/habitats/forest",
            "name": {"en": "Forest", "es": "Bosque"}
          },
          {
            "id": "https://example.org/env/habitats/wetland",
            "name": {"en": "Wetland", "es": "Humedal"}
          },
          {
            "id": "https://example.org/env/habitats/marine",
            "name": {"en": "Marine", "es": "Marino"}
          }
        ]
      },
      {
        "id": "https://example.org/env/protected",
        "name": {"en": "Protected Habitats", "es": "Hábitats Protegidos"},
        "type": "Collection",
        "members": [
          {
            "id": "https://example.org/env/habitats/forest",
            "name": {"en": "Forest", "es": "Bosque"}
          },
          {
            "id": "https://example.org/env/habitats/wetland",
            "name": {"en": "Wetland", "es": "Humedal"}
          }
        ]
      }
    ]
  },
  {
    "id": "https://example.org/species",
    "name": "Species Registry",
    "items": [
      {
        "id": "https://example.org/species/mammals",
        "name": "Mammals",
        "definition": "Registry of mammal species.",
        "concepts": [
          {
            "id": "https://example.org/species/mammals/bear",
            "name": "Bear"
          },
          {
            "id": "https://example.org/species/mammals/wolf",
            "name": "Wolf"
          }
        ]
      }
    ]
  }
]

```

#### jsonld
```jsonld
{
  "@context": [
    {
      "ex": "https://example.org/"
    },
    "https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/hierarchy/default/context.jsonld"
  ],
  "@graph": [
    {
      "id": "https://example.org/env",
      "name": {
        "en": "Environmental Data",
        "es": "Datos Ambientales"
      },
      "items": [
        {
          "id": "https://example.org/env/habitats",
          "name": {
            "en": "Habitat Types",
            "es": "Tipos de H\u00e1bitat"
          },
          "definition": "Classification of natural habitats.",
          "concepts": [
            {
              "id": "https://example.org/env/habitats/forest",
              "name": {
                "en": "Forest",
                "es": "Bosque"
              },
              "type": "Concept"
            },
            {
              "id": "https://example.org/env/habitats/wetland",
              "name": {
                "en": "Wetland",
                "es": "Humedal"
              },
              "type": "Concept"
            },
            {
              "id": "https://example.org/env/habitats/marine",
              "name": {
                "en": "Marine",
                "es": "Marino"
              },
              "type": "Concept"
            }
          ],
          "type": "ConceptScheme"
        },
        {
          "id": "https://example.org/env/protected",
          "name": {
            "en": "Protected Habitats",
            "es": "H\u00e1bitats Protegidos"
          },
          "type": "Collection",
          "members": [
            {
              "id": "https://example.org/env/habitats/forest",
              "name": {
                "en": "Forest",
                "es": "Bosque"
              },
              "type": "Concept"
            },
            {
              "id": "https://example.org/env/habitats/wetland",
              "name": {
                "en": "Wetland",
                "es": "Humedal"
              },
              "type": "Concept"
            }
          ]
        }
      ],
      "type": "Catalog"
    },
    {
      "id": "https://example.org/species",
      "name": "Species Registry",
      "items": [
        {
          "id": "https://example.org/species/mammals",
          "name": "Mammals",
          "definition": "Registry of mammal species.",
          "concepts": [
            {
              "id": "https://example.org/species/mammals/bear",
              "name": "Bear",
              "type": "Concept"
            },
            {
              "id": "https://example.org/species/mammals/wolf",
              "name": "Wolf",
              "type": "Concept"
            }
          ],
          "type": "ConceptScheme"
        }
      ],
      "type": "Catalog"
    }
  ]
}
```

#### ttl
```ttl
@prefix dcat: <http://www.w3.org/ns/dcat#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix ex: <https://example.org/> .
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .

ex:env a dcat:Catalog ;
    dcterms:hasPart <https://example.org/env/habitats>,
        <https://example.org/env/protected> ;
    skos:prefLabel "Environmental Data"@en,
        "Datos Ambientales"@es .

ex:species a dcat:Catalog ;
    dcterms:hasPart <https://example.org/species/mammals> ;
    skos:prefLabel "Species Registry" .

<https://example.org/env/habitats/marine> a skos:Concept ;
    skos:inScheme <https://example.org/env/habitats> ;
    skos:prefLabel "Marine"@en,
        "Marino"@es ;
    skos:topConceptOf <https://example.org/env/habitats> .

<https://example.org/env/protected> a skos:Collection ;
    skos:member <https://example.org/env/habitats/forest>,
        <https://example.org/env/habitats/wetland> ;
    skos:prefLabel "Protected Habitats"@en,
        "Hábitats Protegidos"@es .

<https://example.org/species/mammals/bear> a skos:Concept ;
    skos:inScheme <https://example.org/species/mammals> ;
    skos:prefLabel "Bear" ;
    skos:topConceptOf <https://example.org/species/mammals> .

<https://example.org/species/mammals/wolf> a skos:Concept ;
    skos:inScheme <https://example.org/species/mammals> ;
    skos:prefLabel "Wolf" ;
    skos:topConceptOf <https://example.org/species/mammals> .

<https://example.org/env/habitats/forest> a skos:Concept ;
    skos:inScheme <https://example.org/env/habitats> ;
    skos:prefLabel "Forest"@en,
        "Bosque"@es ;
    skos:topConceptOf <https://example.org/env/habitats> .

<https://example.org/env/habitats/wetland> a skos:Concept ;
    skos:inScheme <https://example.org/env/habitats> ;
    skos:prefLabel "Wetland"@en,
        "Humedal"@es ;
    skos:topConceptOf <https://example.org/env/habitats> .

<https://example.org/species/mammals> a skos:ConceptScheme ;
    skos:definition "Registry of mammal species." ;
    skos:hasTopConcept <https://example.org/species/mammals/bear>,
        <https://example.org/species/mammals/wolf> ;
    skos:prefLabel "Mammals" .

<https://example.org/env/habitats> a skos:ConceptScheme ;
    skos:definition "Classification of natural habitats." ;
    skos:hasTopConcept <https://example.org/env/habitats/forest>,
        <https://example.org/env/habitats/marine>,
        <https://example.org/env/habitats/wetland> ;
    skos:prefLabel "Habitat Types"@en,
        "Tipos de Hábitat"@es .


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
          $ref: '#/$defs/SecondLevelEntry'
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
    required:
    - type
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
  SecondLevelEntry:
    allOf:
    - $ref: '#/$defs/NamedResource'
    oneOf:
    - $ref: '#/$defs/ConceptScheme'
    - $ref: '#/$defs/Collection'
    - required:
      - type
      properties:
        type:
          enum:
          - Dataset
          - Resource
          x-jsonld-id: '@type'
anyOf:
- $ref: '#/$defs/Catalog'
- type: array
  items:
    $ref: '#/$defs/Catalog'
- type: object
  required:
  - catalogs
  properties:
    base:
      type: string
      format: uri
      x-jsonld-id: '@base'
    catalogs:
      type: array
      items:
        $ref: '#/$defs/Catalog'
      x-jsonld-id: '@nest'
x-jsonld-extra-terms:
  id: '@id'
  name:
    x-jsonld-id: http://www.w3.org/2004/02/skos/core#prefLabel
    x-jsonld-container: '@language'
  definition:
    x-jsonld-id: http://www.w3.org/2004/02/skos/core#definition
    x-jsonld-container: '@language'
  Catalog: http://www.w3.org/ns/dcat#Catalog
  ConceptScheme: http://www.w3.org/2004/02/skos/core#ConceptScheme
  Collection: http://www.w3.org/2004/02/skos/core#Collection
  Concept: http://www.w3.org/2004/02/skos/core#Concept
  Dataset: http://www.w3.org/ns/dcat#Dataset
  Resource: http://www.w3.org/ns/dcat#Resource
  members:
    x-jsonld-id: http://www.w3.org/2004/02/skos/core#member
    x-jsonld-type: '@id'
  concepts:
    x-jsonld-reverse: skos:inScheme
x-jsonld-prefixes:
  skos: http://www.w3.org/2004/02/skos/core#
  dcat: http://www.w3.org/ns/dcat#
  dct: http://purl.org/dc/terms/

```

Links to the schema:

* YAML version: [schema.yaml](https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/hierarchy/default/schema.json)
* JSON version: [schema.json](https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/hierarchy/default/schema.yaml)


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
    "base": "@base",
    "catalogs": "@nest",
    "Catalog": "dcat:Catalog",
    "ConceptScheme": "skos:ConceptScheme",
    "Collection": "skos:Collection",
    "Concept": "skos:Concept",
    "Dataset": "dcat:Dataset",
    "Resource": "dcat:Resource",
    "members": {
      "@id": "skos:member",
      "@type": "@id"
    },
    "concepts": {
      "@reverse": "skos:inScheme"
    },
    "skos": "http://www.w3.org/2004/02/skos/core#",
    "dcat": "http://www.w3.org/ns/dcat#",
    "dct": "http://purl.org/dc/terms/",
    "@version": 1.1
  }
}
```

You can find the full JSON-LD context here:
[context.jsonld](https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/hierarchy/default/context.jsonld)


# For developers

The source code for this Building Block can be found in the following repository:

* URL: [https://github.com/ogcincubator/bblocks-prez](https://github.com/ogcincubator/bblocks-prez)
* Path: `_sources/hierarchy/default`

