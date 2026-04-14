---
title: Data-Mapped Profile (RDF Generation)
weight: 2
toc: true
---

A YAMAML document with data mappings that generates RDF from a CSV file.

This example illustrates the `defaults.mapping` shortcut for sharing a data source across all statements, the `id.mapping.path` pattern for deriving record identifiers from a column, and the use of `prepend` and `separator` transformations on individual statement mappings.

## YAMAML

```yaml
%YAML 1.2
---
base: http://example.org/people/

namespaces:
  foaf: http://xmlns.com/foaf/0.1/
  schema: http://schema.org/
  xsd: http://www.w3.org/2001/XMLSchema#
  rdfs: http://www.w3.org/2000/01/rdf-schema#

defaults:
  mapping:
    source: people.csv
    type: csv

descriptions:
  person:
    a: foaf:Person
    label: Person
    note: A person record
    id:
      mapping:
        path: ID

    statements:
      name:
        label: Name
        property: foaf:name
        min: 1
        max: 1
        type: literal
        datatype: xsd:string
        mapping:
          path: name

      email:
        label: Email
        property: foaf:mbox
        min: 0
        type: IRI
        mapping:
          path: email
          prepend: "mailto:"

      knows:
        label: Knows
        property: foaf:knows
        min: 0
        type: IRI
        description: person
        mapping:
          path: friends
          separator: ","

      address:
        label: Address
        property: schema:address
        min: 0
        max: 1
        type: BNODE
        description: postalAddress

  postalAddress:
    a: schema:PostalAddress
    label: Address
    statements:
      street:
        label: Street
        property: schema:streetAddress
        type: literal
        datatype: xsd:string
        mapping:
          path: street

      city:
        label: City
        property: schema:addressLocality
        type: literal
        datatype: xsd:string
        mapping:
          path: city

      postalCode:
        label: Postal Code
        property: schema:postalCode
        type: literal
        datatype: xsd:string
        mapping:
          path: zip
```

## See also

The same profile authored in PKL is shown in [PKL §10.2 Data-mapped profile](../../pkl#102-data-mapped-profile-rdf-generation).
