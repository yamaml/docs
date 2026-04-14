---
title: Rich Constraints
weight: 3
toc: true
---

A YAMAML document demonstrating the full range of statement-level constraints: datatypes, numeric and string facets, regex patterns, picklists (value lists), and closed shapes.

## YAMAML

```yaml
%YAML 1.2
---
base: http://example.org/products/

namespaces:
  schema: http://schema.org/
  xsd: http://www.w3.org/2001/XMLSchema#

descriptions:
  product:
    a: schema:Product
    label: Product
    closed: true
    statements:
      name:
        label: Name
        property: schema:name
        min: 1
        max: 1
        type: literal
        datatype: xsd:string

      price:
        label: Price
        property: schema:price
        min: 1
        max: 1
        type: literal
        datatype: xsd:decimal
        facets:
          MinInclusive: 0
          MaxInclusive: 999999.99

      sku:
        label: SKU
        property: schema:sku
        min: 1
        max: 1
        type: literal
        datatype: xsd:string
        pattern: "^[A-Z]{3}-\\d{6}$"
        facets:
          Length: 10

      category:
        label: Category
        property: schema:category
        min: 1
        max: 1
        type: literal
        values:
          - electronics
          - clothing
          - books
          - food

      manufacturer:
        label: Manufacturer
        property: schema:manufacturer
        min: 0
        max: 1
        type: IRI
        description: organization

  organization:
    a: schema:Organization
    label: Organization
    statements:
      name:
        label: Name
        property: schema:name
        min: 1
        max: 1
        type: literal
        datatype: xsd:string
      url:
        label: Website
        property: schema:url
        min: 0
        max: 1
        type: IRI
```

## See also

The same profile authored in PKL is shown in [PKL §10.3 Rich constraints](../../pkl#103-rich-constraints).
