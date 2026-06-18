
# Namespace prefix declaration (Schema)

`ogc.prez.prefix` *v0.1*

Declares a preferred namespace prefix and the URI it expands to, for use in CURIE generation.

[*Status*](http://www.opengis.net/def/status): Under development

## Description

This building block defines a JSON input format for declaring a preferred namespace prefix and the
URI it expands to, following the [VANN](http://vocab.org/vann/) vocabulary
(`vann:preferredNamespacePrefix` / `vann:preferredNamespaceUri`).

A prefix declaration has no `id`: it uplifts to a standalone RDF node (a blank node), not linked to
any other resource. [Prez](https://github.com/RDFLib/prez) scans the dataset graph for these
declarations to build its global CURIE map, so they are typically mixed alongside catalogs rather
than nested under them — see the [default hierarchy](bblocks://ogc.prez.hierarchy.default) block.
## Examples

### Registers collection prefix
Declares the `registers` prefix for the OGC catalog register collections namespace.

#### json
```json
{
  "prefix": "registers",
  "uri": "urn:ogc:defs/catalogs/register/collections/"
}
```

#### jsonld
```jsonld
{
  "@context": "https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/prefix/context.jsonld",
  "prefix": "registers",
  "uri": "urn:ogc:defs/catalogs/register/collections/"
}
```

#### ttl
```ttl
@prefix vann: <http://purl.org/vocab/vann/> .

[] vann:preferredNamespacePrefix "registers" ;
    vann:preferredNamespaceUri "urn:ogc:defs/catalogs/register/collections/" .


```

## Schema

```yaml
$schema: https://json-schema.org/draft/2020-12/schema
title: Namespace prefix declaration
description: 'Declares a preferred namespace prefix and the URI it expands to, following
  the VANN vocabulary

  (`vann:preferredNamespacePrefix` / `vann:preferredNamespaceUri`).

  '
type: object
required:
- prefix
- uri
properties:
  prefix:
    type: string
    pattern: ^[a-zA-Z][a-zA-Z0-9_-]*$
    x-jsonld-id: http://purl.org/vocab/vann/preferredNamespacePrefix
  uri:
    type: string
    format: uri
    x-jsonld-id: http://purl.org/vocab/vann/preferredNamespaceUri
additionalProperties: false
x-jsonld-prefixes:
  vann: http://purl.org/vocab/vann/

```

Links to the schema:

* YAML version: [schema.yaml](https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/prefix/schema.json)
* JSON version: [schema.json](https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/prefix/schema.yaml)


# JSON-LD Context

```jsonld
{
  "@context": {
    "prefix": "vann:preferredNamespacePrefix",
    "uri": "vann:preferredNamespaceUri",
    "vann": "http://purl.org/vocab/vann/",
    "@version": 1.1
  }
}
```

You can find the full JSON-LD context here:
[context.jsonld](https://ogcincubator.github.io/bblocks-prez/build/annotated/prez/prefix/context.jsonld)


# For developers

The source code for this Building Block can be found in the following repository:

* URL: [https://github.com/ogcincubator/bblocks-prez](https://github.com/ogcincubator/bblocks-prez)
* Path: `_sources/prefix`

