This building block defines a JSON input format for declaring a preferred namespace prefix and the
URI it expands to, following the [VANN](http://vocab.org/vann/) vocabulary
(`vann:preferredNamespacePrefix` / `vann:preferredNamespaceUri`).

A prefix declaration has no `id`: it uplifts to a standalone RDF node (a blank node), not linked to
any other resource. [Prez](https://github.com/RDFLib/prez) scans the dataset graph for these
declarations to build its global CURIE map, so they are typically mixed alongside catalogs rather
than nested under them — see the [default hierarchy](bblocks://ogc.prez.hierarchy.default) block.