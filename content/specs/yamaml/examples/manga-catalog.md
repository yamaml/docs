---
title: Metadata Profile (No Data Mapping)
weight: 1
toc: true
---

A YAMAML document describing a manga catalog schema with multiple linked descriptions.

This example shows the core authoring style: `base` URI, namespace declarations, and a `descriptions` map where each description references other descriptions by name in its statement `description` fields.

## YAMAML

```yaml
%YAML 1.2
---
base: http://example.org/manga/

namespaces:
  schema: http://schema.org/
  xsd: http://www.w3.org/2001/XMLSchema#
  dcterms: http://purl.org/dc/terms/
  foaf: http://xmlns.com/foaf/0.1/

descriptions:
  series:
    a: schema:ComicSeries
    label: Manga Series
    note: A manga series or title
    statements:
      title:
        label: Title
        property: schema:name
        min: 1
        max: 1
        type: literal
        datatype: xsd:string
      genre:
        label: Genre
        property: schema:genre
        min: 1
        type: literal
        datatype: xsd:string
      author:
        label: Author
        property: schema:author
        min: 1
        type: IRI
        description: creator
      volume:
        label: Volumes
        property: schema:hasPart
        min: 1
        type: IRI
        description: volume

  volume:
    a: schema:Book
    label: Volume
    note: A single volume of a manga series
    statements:
      volumeNumber:
        label: Volume Number
        property: schema:volumeNumber
        min: 1
        max: 1
        type: literal
        datatype: xsd:integer
      datePublished:
        label: Date Published
        property: schema:datePublished
        min: 1
        max: 1
        type: literal
        datatype: xsd:date

  creator:
    a: foaf:Person
    label: Creator
    note: A manga author or artist
    statements:
      name:
        label: Name
        property: schema:name
        min: 1
        max: 1
        type: literal
        datatype: xsd:string
      birthDate:
        label: Birth Date
        property: schema:birthDate
        min: 0
        max: 1
        type: literal
        datatype: xsd:date
```

## See also

The same profile authored in PKL is shown in [PKL §10.1 Metadata profile](../../pkl#101-metadata-profile-no-data-mapping).
