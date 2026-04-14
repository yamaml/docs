---
title: YAMAML Specification
weight: 1
toc: true
---

**Version:** 0.2.0
**Status:** Working Draft
**Date:** 2026-03-14
**Latest:** https://docs.yamaml.org/specs/yamaml/spec/

## Abstract

YAMAML (Yet Another Metadata Application Profile and RDF Mapping Language) is a human-friendly markup language for creating, managing, and publishing metadata application profiles. YAMAML documents describe the structure and constraints of metadata records using YAML syntax, and can be transformed into standard formats including RDF, SHACL, ShEx, OWL-DSP, SimpleDSP, DCTAP, and Frictionless Data Packages.

## Status of This Document

This document defines the textual syntax and semantics of YAMAML. It is a working draft and subject to change.

## Historical Note

YAMAML is based on the [YAMA specification](http://purl.org/yama/spec/latest) (Yet Another Metadata Application Profile, circa 2019). YAMAML extends and refines the original YAMA concepts with a clearer document model, explicit data mapping capabilities, richer constraint vocabulary, and interoperability with modern metadata standards. Where the original YAMA specification used the term "Description Set" as a top-level container, YAMAML simplifies this to a flat document model. Elements such as `description_set`, `standalone`, `name`, `long_description`, and `constraint` from the original YAMA specification have been superseded by the elements defined in this document.

## 1. Introduction

### 1.1 Design Goals

YAMAML is designed to be:

- **Human-readable** — authored and reviewed in a plain text editor.
- **YAML-native** — parsable by any YAML 1.2 compliant parser.
- **Multi-target** — a single source document produces RDF, SHACL, ShEx, DCTAP, DSP, diagrams, and more.
- **Extensible** — custom keys are permitted alongside the reserved vocabulary defined here.
- **Practical** — includes optional data mapping for generating RDF from tabular or structured data sources.

### 1.2 Syntax Compatibility

YAMAML documents are valid YAML 1.2 documents. A YAMAML document SHOULD begin with the YAML directive and document start marker:

```yaml
%YAML 1.2
---
```

### 1.3 Conventions

The key words "MUST", "SHOULD", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

In property tables:

- **R** = Required
- **Type** = Expected YAML type
- **Default** = Value assumed when the key is absent

---

## 2. Document Structure

A YAMAML document is a YAML mapping with the following top-level structure:

```yaml
%YAML 1.2
---
base: <IRI>
namespaces:
  <prefix>: <IRI>
defaults:
  mapping:
    <mapping-properties>
data:
  - <inline-records>
descriptions:
  <description-name>:
    <description-properties>
```

### 2.1 Top-Level Properties

| Key | Type | Default | Description | Required |
|-----|------|---------|-------------|----------|
| `base` | IRI (string) | — | Base IRI for the application profile. Used as the namespace for description and statement IRIs in generated outputs. | — |
| `namespaces` | Mapping | — | Prefix-to-IRI namespace declarations. | — |
| `defaults` | Mapping | — | Default values inherited by all descriptions and statements. Currently supports `mapping` defaults. | — |
| `data` | Sequence or Mapping | — | Inline data records. Referenced by mapping `source: data`. | — |
| `descriptions` | Mapping | — | The set of description definitions that form the application profile. | **R** |

### 2.2 Namespaces

The `namespaces` section declares prefix-to-IRI bindings used throughout the document for compact IRI notation (prefixed names).

```yaml
namespaces:
  foaf: http://xmlns.com/foaf/0.1/
  schema: http://schema.org/
  xsd: http://www.w3.org/2001/XMLSchema#
  dcterms: http://purl.org/dc/terms/
```

#### Prefixed Names

A prefixed name has the form `prefix:localName` (e.g., `foaf:name`). Processors expand prefixed names by replacing `prefix:` with the corresponding namespace IRI. Full IRIs (`http://...`, `urn:...`) are also accepted wherever a prefixed name is expected.

#### Standard Prefixes

A YAMAML processor maintains a standard prefix table identical to the [SimpleDSP table](/specs/simpledsp/spec/#7-standard-namespace-prefixes-626): `dc`, `dcterms`, `foaf`, `skos`, `xl`, `rdf`, `rdfs`, `owl`, `xsd`, and `schema`. These entries are consulted as a fallback to resolve any prefixed name whose prefix is not declared in the document's `namespaces` section.

#### Resolution Rule

For any prefixed name `prefix:localName` in a YAMAML document, a YAMAML processor resolves the prefix as follows:

1. If `prefix` is declared in `namespaces`, use the IRI bound there.
2. Otherwise, if `prefix` appears in the standard prefix table, use the IRI from the table.
3. Otherwise, the prefix is undeclared and the document is not well-formed.

The user's `namespaces` declarations take precedence unconditionally. A prefix declared in `namespaces` may bind to any IRI; the processor does not compare it against the standard table. Authors may redefine a standard prefix, use unconventional labels (for example, `DCTERMS` instead of `dcterms`), or bind a familiar label to a different IRI for local conventions. Prefix labels in RDF/SHACL/ShEx outputs mirror those used in the profile — exported `@prefix` declarations are derived from the resolved bindings, not renamed to canonical forms.

If every prefixed name in the document resolves through the standard prefix table, the `namespaces` section may be omitted.

### 2.3 Defaults

The `defaults` section provides inherited values for descriptions and their statements, reducing repetition when many statements share common properties.

```yaml
defaults:
  mapping:
    source: data.csv
    type: csv
```

Currently, only `mapping` defaults are defined. When a statement's `mapping` is specified, its properties are merged with the defaults — statement-level values override defaults.

### 2.4 Inline Data

The `data` section holds inline records that can be used as a data source via `source: data` in a mapping. This is an alternative to referencing external files.

```yaml
data:
  - id: 1
    name: Alice
  - id: 2
    name: Bob
```

---

## 3. Descriptions

A **description** defines a class of resources — the equivalent of a shape, template, or record type. Each description is identified by its key within the `descriptions` mapping.

```yaml
descriptions:
  book:
    a: schema:Book
    label: Book
    note: A published book
    statements:
      title:
        property: dcterms:title
        # ...
```

### 3.1 Description Properties

| Key | Type | Default | Description | Required |
|-----|------|---------|-------------|----------|
| `a` | Prefixed name or IRI | — | The RDF class (`rdf:type`) of the described resource. | — |
| `label` | String | — | Human-readable label for the description. Used in SHACL (`sh:name`), DSP (`rdfs:label`), DCTAP (`shapeLabel`), and diagrams. | — |
| `note` | String | — | Documentation or explanatory text. Used in SHACL (`sh:description`), DSP (`rdfs:comment`), DCTAP (`note`). | — |
| `id` | Mapping | — | Identifier configuration for data-mapped descriptions. See [Section 3.2](#32-identifier-mapping). | — |
| `statements` | Mapping | — | The set of statement definitions for this description. See [Section 4](#4-statements). | **R** |
| `closed` | Boolean | `false` | When `true`, the description is treated as a closed shape in SHACL (`sh:closed true`), meaning only the declared properties are permitted. | — |

### 3.2 Identifier Mapping

Descriptions that participate in RDF generation from data sources require an `id` section that specifies how record identifiers are extracted from the data.

```yaml
descriptions:
  character:
    a: foaf:Person
    id:
      prefix: tbbt
      mapping:
        source: characters.csv
        type: csv
        path: ID
```

#### `id` Properties

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `prefix` | String | — | A namespace prefix declared in `namespaces` that provides the base URI for record identifiers. The identifier value is appended to the namespace URI to form the subject IRI. For example, if `prefix: ndlbooks` and the namespace maps `ndlbooks` to `http://iss.ndl.go.jp/books/`, then a record with ID `b01234` gets the subject IRI `http://iss.ndl.go.jp/books/b01234`. |
| `mapping` | Mapping | — | Data source mapping. Follows the same structure as statement-level mappings (see [Section 5](#5-data-mapping)). The `path` field identifies the column or field that provides the unique identifier for each record. |

When `prefix` is omitted, the identifier value is appended to the document `base` to form the subject IRI (backward-compatible behavior). When `prefix` is present, the referenced namespace takes precedence over `base` for that description's records.

In SimpleDSP, `prefix` corresponds to the ID row's value constraint column — a namespace prefix ending with `:` (e.g. `ndlbooks:`). The document-level `base` corresponds to `@base` in the SimpleDSP `[@NS]` block, which is the schema namespace (where the description template definitions live), distinct from the record namespace.

```yaml
# Schema namespace — where the DSP definitions live
base: http://ndl.go.jp/dcndl/dsp/biblio

namespaces:
  ndlbooks: http://iss.ndl.go.jp/books/
  ndlauthors: http://iss.ndl.go.jp/authors/

descriptions:
  MAIN:
    a: foaf:Document
    id:
      prefix: ndlbooks       # records live at http://iss.ndl.go.jp/books/{id}
      mapping:
        path: 書誌ID

  Author:
    a: foaf:Person
    id:
      prefix: ndlauthors     # records live at http://iss.ndl.go.jp/authors/{id}
      mapping:
        path: AuthorID
```

### 3.3 Description References

Descriptions can reference other descriptions to model relationships:

- **IRI references** — A statement with `type: IRI` and `description: <name>` creates a typed relationship to another description.
- **Blank node references** — A statement with `type: BNODE` and `description: <name>` creates an inline blank node with the properties of the referenced description.

```yaml
descriptions:
  person:
    a: foaf:Person
    statements:
      address:
        property: schema:address
        type: BNODE
        description: postalAddress    # inline blank node
      knows:
        property: foaf:knows
        type: IRI
        description: person           # self-reference (IRI link)

  postalAddress:
    a: schema:PostalAddress
    statements:
      street:
        property: schema:streetAddress
        type: literal
        datatype: xsd:string
```

---

## 4. Statements

A **statement** defines a single property of a described resource — its predicate, expected value type, cardinality constraints, and optional data mapping.

```yaml
statements:
  title:
    label: Title
    property: dcterms:title
    min: 1
    max: 1
    type: literal
    datatype: xsd:string
    note: The title of the resource
```

### 4.1 Statement Properties

| Key | Type | Default | Description | Required |
|-----|------|---------|-------------|----------|
| `property` | Prefixed name or IRI | — | The RDF property (predicate). | **R** |
| `label` | String | — | Human-readable label. Used in SHACL (`sh:name`), DSP (`rdfs:label`), DCTAP (`propertyLabel`), SimpleDSP (`ItemRuleName`). | — |
| `note` | String | — | Documentation text. Used in SHACL (`sh:description`), DSP (`rdfs:comment`), DCTAP/SimpleDSP (`Comment`). | — |
| `type` | String | `literal` | Value node type. See [Section 4.2](#42-value-types). | — |
| `datatype` | Prefixed name or IRI | — | XSD datatype constraint (e.g., `xsd:string`, `xsd:date`). Applicable when `type` is `literal`. | — |
| `min` | Integer | — | Minimum cardinality. `0` = optional, `1` = required. | — |
| `max` | Integer | — | Maximum cardinality. Absence means unbounded. | — |
| `description` | String | — | Name of another description this statement references. Semantics depend on `type`: `BNODE` creates blank nodes; `IRI` creates typed links. | — |
| `values` | Sequence | — | Enumerated value set (picklist). For literals, these are allowed string values. For IRIs, these are allowed URI values. | — |
| `pattern` | String | — | Regular expression constraint on the value. | — |
| `facets` | Mapping | — | Numeric and string facets. See [Section 4.4](#44-facets). | — |
| `mapping` | Mapping | — | Data source mapping. See [Section 5](#5-data-mapping). | — |
| `inScheme` | String or Sequence | — | Vocabulary scheme reference(s). Used in OWL-DSP (`dsp:inScheme`). A scheme prefix ending with `:` denotes a namespace constraint. | — |
| `cardinalityNote` | String | — | Descriptive cardinality keyword (e.g., recommended, conditional). Preserved in OWL-DSP as `dsp:cardinalityNote`. When present, `min` and `max` MAY be absent. | — |

### 4.2 Value Types

The `type` property declares the kind of value a statement expects.

| Value | Description |
|-------|-------------|
| `literal` | A literal (text) value. May be further constrained by `datatype`. This is the default when `type` is omitted and `datatype` is present. |
| `IRI` | A URI/IRI reference. When combined with `description`, creates a link to another described resource. |
| `URI` | Alias for `IRI`. |
| `BNODE` | A blank node. MUST be combined with `description` to define the blank node's structure. |

### 4.3 Cardinality

Cardinality is expressed through `min` and `max`:

| min | max | Meaning |
|-----|-----|---------|
| `0` | `1` | Optional, at most once |
| `1` | `1` | Required, exactly once |
| `1` | *(absent)* | Required, repeatable |
| `0` | *(absent)* | Optional, repeatable |
| *(absent)* | *(absent)* | No constraint specified |

When `max` is absent, the property is unbounded (repeatable without limit).

The `cardinalityNote` property allows descriptive cardinality that cannot be expressed as numeric min/max, such as "recommended" or "mandatory if applicable". When `cardinalityNote` is present:
- `min` and `max` MAY still be specified for machine-processable cardinality.
- Processors that support descriptive cardinality (e.g., OWL-DSP) SHOULD emit the note alongside any numeric constraints.

### 4.4 Facets

The `facets` mapping provides fine-grained constraints on values, following XSD facet semantics:

| Facet | Type | Description |
|-------|------|-------------|
| `MinInclusive` | Number | Minimum value (inclusive) |
| `MaxInclusive` | Number | Maximum value (inclusive) |
| `MinExclusive` | Number | Minimum value (exclusive) |
| `MaxExclusive` | Number | Maximum value (exclusive) |
| `MinLength` | Integer | Minimum string length |
| `MaxLength` | Integer | Maximum string length |
| `Length` | Integer | Exact string length |
| `TotalDigits` | Integer | Maximum total digits |
| `FractionDigits` | Integer | Maximum fraction digits |

```yaml
statements:
  age:
    property: foaf:age
    type: literal
    datatype: xsd:integer
    facets:
      MinInclusive: 0
      MaxInclusive: 150
  isbn:
    property: schema:isbn
    type: literal
    datatype: xsd:string
    facets:
      Length: 13
```

### 4.5 Value Constraints

#### Enumerated Values

The `values` property restricts a statement to a fixed set of allowed values:

```yaml
# Literal picklist
format:
  property: dcterms:format
  type: literal
  values:
    - print
    - ebook
    - audiobook

# IRI value set
license:
  property: dcterms:license
  type: IRI
  values:
    - http://creativecommons.org/licenses/by/4.0/
    - http://creativecommons.org/publicdomain/zero/1.0/
```

#### Pattern

The `pattern` property constrains values using a regular expression:

```yaml
issn:
  property: schema:issn
  type: literal
  datatype: xsd:string
  pattern: "^\\d{4}-\\d{3}[\\dX]$"
```

#### Vocabulary Scheme

The `inScheme` property constrains IRI values to specific vocabulary namespaces:

```yaml
subject:
  property: dcterms:subject
  type: IRI
  inScheme: skos:       # values must be from the SKOS namespace

classification:
  property: dcterms:subject
  type: IRI
  inScheme:             # values from either namespace
    - ndlsh:
    - lcsh:
```

---

## 5. Data Mapping

YAMAML supports an optional data mapping layer for generating RDF from tabular or structured data sources. Mappings are defined at both the description level (for record identity) and the statement level (for property values).

### 5.1 Mapping Properties

| Key | Type | Default | Description | Required |
|-----|------|---------|-------------|----------|
| `source` | String | *(from defaults)* | Path to the data source file, URL, or `"data"` for inline data. | **R** |
| `type` | String | *(inferred from extension)* | Source format: `csv`, `xlsx`, `json`, `yaml`, `yml`. | — |
| `path` | String | — | Field name or JSONata expression for extracting the value from each record. | **R** |
| `strip` | Sequence | — | Characters to remove from extracted values. | — |
| `replace` | Sequence of pairs | — | Substitution pairs `[from, to]` applied in order. | — |
| `separator` | String | — | Character to split a multi-valued field into separate values. | — |
| `prepend` | String | — | String prepended to each extracted value. | — |
| `append` | String | — | String appended to each extracted value. | — |

### 5.2 Data Sources

YAMAML supports the following data source formats:

| Format | Extension | Description |
|--------|-----------|-------------|
| CSV | `.csv` | Comma-separated values. First row is treated as headers. |
| Excel | `.xlsx`, `.xls` | Microsoft Excel workbook. First sheet is used; first row is treated as headers. |
| JSON | `.json` | JSON object or array of objects. |
| YAML | `.yaml`, `.yml` | YAML document or sequence. |
| Inline | `"data"` | Records from the document's `data` section. |
| URL | `http://...` | Remote file fetched over HTTP(S). |

Source paths are resolved relative to the YAMAML document's directory. Absolute paths and URLs are used as-is.

### 5.3 Value Transformation Pipeline

When a mapping is present, extracted values pass through a transformation pipeline in the following order:

1. **Extract** — The `path` expression is evaluated against the data source to obtain the raw value.
2. **Split** — If `separator` is defined and the value is a string, it is split into multiple values.
3. **Strip** — Characters listed in `strip` are removed from each value.
4. **Replace** — Each `[from, to]` pair in `replace` is applied as a global string replacement.
5. **Decorate** — `prepend` and `append` are added to each value.

### 5.4 Defaults and Inheritance

The `defaults.mapping` section provides base mapping properties inherited by all statements. Statement-level mapping properties override defaults:

```yaml
defaults:
  mapping:
    source: characters.csv
    type: csv

descriptions:
  character:
    id:
      mapping:
        path: ID            # inherits source and type from defaults
    statements:
      name:
        property: foaf:name
        mapping:
          path: name         # inherits source and type from defaults
      wikidata:
        property: rdfs:seeAlso
        type: IRI
        mapping:
          source: external.csv   # overrides default source
          path: wikidata_id
          prepend: http://www.wikidata.org/entity/
```

---

## 6. Worked Examples

Complete YAMAML profiles are available in the [Examples](../examples/) section:

{{< cards >}}
  {{< card link="../examples/manga-catalog" title="Metadata Profile" subtitle="A multi-description profile with linked shapes" icon="book-open" >}}
  {{< card link="../examples/people" title="Data-Mapped Profile" subtitle="RDF generation from a CSV file using data mappings" icon="document-text" >}}
  {{< card link="../examples/products" title="Rich Constraints" subtitle="Facets, patterns, picklists, and closed shapes" icon="table" >}}
{{< /cards >}}

The same examples authored in PKL appear in [§10 of the PKL authoring guide](../pkl#10-complete-examples).

---

## 7. Output Format Mappings

This section defines how YAMAML elements map to each supported output format.

### 7.1 RDF

RDF generation requires data mappings. Each record in the data source produces a set of quads.

| YAMAML | RDF |
|--------|-----|
| `base` + record ID | Subject IRI |
| `description.a` | `rdf:type` triple |
| `statement.property` | Predicate IRI |
| `statement.type = IRI` | Object is a NamedNode |
| `statement.type = literal` | Object is a Literal |
| `statement.type = BNODE` | Object is a BlankNode (with nested triples) |
| `statement.datatype` | Literal datatype IRI |

### 7.2 SHACL

| YAMAML | SHACL |
|--------|-------|
| description | `sh:NodeShape` |
| `description.a` | `sh:targetClass` |
| `description.label` | `sh:name` |
| `description.note` | `sh:description` |
| `description.closed` | `sh:closed true` + `sh:ignoredProperties (rdf:type)` |
| statement | `sh:PropertyShape` (via `sh:property`) |
| `statement.property` | `sh:path` |
| `statement.label` | `sh:name` |
| `statement.note` | `sh:description` |
| `statement.min` | `sh:minCount` |
| `statement.max` | `sh:maxCount` |
| `statement.datatype` | `sh:datatype` |
| `statement.type = IRI` | `sh:nodeKind sh:IRI` |
| `statement.type = literal` | `sh:nodeKind sh:Literal` |
| `statement.type = BNODE` | `sh:nodeKind sh:BlankNodeOrIRI` |
| `statement.description` | `sh:node` |
| `statement.values` | `sh:in` |
| `statement.pattern` | `sh:pattern` |
| `statement.facets.MinInclusive` | `sh:minInclusive` |
| `statement.facets.MaxInclusive` | `sh:maxInclusive` |

### 7.3 ShEx

| YAMAML | ShEx Compact Syntax |
|--------|-----|
| description | Shape `<name> { ... }` |
| `description.a` | `EXTRA a` + `a [class]` |
| statement | TripleConstraint |
| `statement.property` | Predicate |
| `statement.type` | Node constraint (`IRI`, `LITERAL`, `BNODE`) |
| `statement.datatype` | Datatype constraint |
| `statement.min/max` | Cardinality (`*`, `+`, `?`, `{m,n}`) |
| `statement.description` | Shape reference `@<shape>` |
| `statement.values` | Value set `["a" "b"]` |
| `statement.pattern` | String facet `//pattern//` |
| `statement.facets.*` | Numeric/string facets |

### 7.4 OWL-DSP

| YAMAML | OWL-DSP |
|--------|---------|
| description | `dsp:DescriptionTemplate` (OWL Class) |
| `description.a` | `dsp:resourceClass` |
| `description.label` | `rdfs:label` |
| `description.note` | `rdfs:comment` |
| `description.id` | `dsp:valueURIOccurrence "mandatory"` |
| statement | `dsp:StatementTemplate` (OWL Restriction) |
| `statement.property` | `owl:onProperty` |
| `statement.min/max` | `owl:minQualifiedCardinality` / `owl:maxQualifiedCardinality` |
| `statement.datatype` | `owl:onDataRange` |
| `statement.description` | `owl:onClass` (shape reference) |
| `statement.inScheme` | `dsp:inScheme` |
| `statement.values` | `owl:oneOf` |
| `statement.cardinalityNote` | `dsp:cardinalityNote` |

### 7.5 SimpleDSP

| YAMAML | SimpleDSP Column |
|--------|-----------------|
| statement key/label | ItemRuleName (項目規則名) |
| `statement.property` | Property (プロパティ) |
| `statement.min` | Min (最小) |
| `statement.max` (absent = `-`) | Max (最大) |
| value type (see below) | Value type (値タイプ) |
| constraint (see below) | Constraint (値制約) |
| `statement.note` | Comment (コメント) |

SimpleDSP value type resolution:

| Condition | SimpleDSP Value Type |
|-----------|---------------------|
| Has `description` or `a` (class constraint) | `structured` (構造化) |
| `type` is IRI/URI | `reference` (参照値) |
| `type` is literal, or `datatype` present, or `values` present | `literal` (文字列) |
| None of the above | *(empty)* (制約なし) |

### 7.6 DCTAP

| YAMAML | DCTAP Column |
|--------|--------------|
| description name | `shapeID` |
| `description.label` | `shapeLabel` |
| `statement.property` | `propertyID` |
| `statement.label` | `propertyLabel` |
| `statement.min >= 1` | `mandatory = TRUE` |
| `statement.max` absent or > 1 | `repeatable = TRUE` |
| `statement.type` | `valueNodeType` |
| `statement.datatype` | `valueDataType` |
| `statement.values` | `valueConstraint` (picklist) |
| `statement.pattern` | `valueConstraint` (pattern) |
| `statement.description` | `valueShape` |
| `statement.note` | `note` |

### 7.7 Frictionless Data Package

| YAMAML | Data Package |
|--------|-------------|
| `base` | `package.id` |
| data source | Resource |
| `description.label` | `resource.title` |
| `description.note` | `resource.description` |
| `id.mapping.path` | `schema.primaryKey` |
| `statement.mapping.path` | `field.name` |
| `statement.label` | `field.title` |
| `statement.note` | `field.description` |
| `statement.datatype` | `field.type` (XSD → Frictionless mapping) |
| `statement.type = IRI` | `field.type = "string"`, `field.format = "uri"` |
| `statement.min >= 1` | `field.constraints.required = true` |
| `statement.values` | `field.constraints.enum` |
| `statement.pattern` | `field.constraints.pattern` |
| `statement.facets.MinInclusive` | `field.constraints.minimum` |
| `statement.facets.MaxInclusive` | `field.constraints.maximum` |

---

## 8. Processing Model

### 8.1 IRI Expansion

Processors MUST expand prefixed names to full IRIs using the following rules:

1. If the term matches `^(https?|urn):`, it is already a full IRI.
2. If the term contains `:`, split on the first `:` to get `prefix` and `localName`. If `prefix` matches a declared namespace, expand to `namespace_IRI + localName`.
3. Otherwise, if `base` is defined, expand to `base + term`.
4. Otherwise, use the term as-is.

### 8.2 Source Path Resolution

Data source paths in mappings are resolved as follows:

1. If the path starts with `http://` or `https://`, it is treated as a URL and fetched over the network.
2. If the path starts with `/`, it is an absolute file path.
3. Otherwise, it is resolved relative to the directory containing the YAMAML document.

### 8.3 Type Inference

When `type` is omitted from a statement:

- If `datatype` is present, the value type is implicitly `literal`.
- If `description` is present and no `type` is specified, the relationship type depends on context (processors MAY default to `IRI`).
- If neither `datatype` nor `description` is present, processors SHOULD treat the type as `literal`.

---

## 9. Conformance

### 9.1 Document Conformance

A conforming YAMAML document:

1. MUST be valid YAML 1.2.
2. MUST contain a `descriptions` mapping with at least one description.
3. Each description MUST contain a `statements` mapping (descriptions without statements are permitted but not actionable).
4. Each statement that participates in output generation MUST have a `property` key.

### 9.2 Processor Conformance

A conforming YAMAML processor:

1. MUST parse YAML 1.2 documents.
2. MUST expand prefixed names using declared namespaces.
3. MUST support the top-level keys defined in [Section 2.1](#21-top-level-properties).
4. MUST support the description properties defined in [Section 3.1](#31-description-properties).
5. MUST support the statement properties defined in [Section 4.1](#41-statement-properties).
6. SHOULD ignore unknown keys without raising an error.
7. MAY support a subset of output formats.

---

## 10. Differences from YAMA 0.1.5

This section summarizes the key differences between YAMAML and the original YAMA specification (v0.1.5, circa 2019).

| Aspect | YAMA 0.1.5 | YAMAML |
|--------|-----------|--------|
| Top-level container | `description_set` with metadata (id, title, version, date, creator, etc.) | Flat document; metadata expressed through `base` and `namespaces` |
| Description key `name` | Separate `name` and `label` | `label` only (serves both purposes) |
| Description key `description` | `description` and `long_description` | `note` (single field) |
| Description key `standalone` | Boolean, default `true` | Removed; inferred from presence of `id` mapping |
| Statement constraints | `constraint: x or [x,y]` referencing external constraint IDs | Inline: `datatype`, `values`, `pattern`, `facets` |
| Data mapping | Not specified | Full mapping section with source, path, transformations |
| Value types | `type` (unspecified values) | `type`: `literal`, `IRI`, `URI`, `BNODE` |
| Namespace handling | Same concept | Same; added standard prefix table for SimpleDSP interop |
| RDF class | Referenced as `class` (marked with X as experimental) | `a` (short for `rdf:type`, following Turtle convention) |
| Closed shapes | Not supported | `closed: true` for SHACL |
| Inline data | Not supported | `data` section + `source: data` |
| URL input | Not supported | HTTP(S) URLs accepted for `-i` and data sources |

---

## References

- YAML 1.2 Specification: https://yaml.org/spec/1.2/spec.html
- YAMA Specification (historical, circa 2019): http://purl.org/yama/spec/latest
- SHACL: https://www.w3.org/TR/shacl/
- ShEx: https://shex.io
- DCTAP: https://dcmi.github.io/dctap/
- OWL-DSP Ontology: https://www.kanzaki.com/ns/dsp#
- SimpleDSP: See the [SimpleDSP Specification](/specs/simpledsp/spec/)
- Frictionless Data Package: https://datapackage.org
- Semantic Versioning: https://semver.org


## Related Publications

The following peer-reviewed publications describe the research and design behind YAMA and YAMAML, listed in chronological order:

1. Thalhath, N., Nagamori, M., Sakaguchi, T., & Sugimoto, S. (2019). **Authoring Formats and Their Extensibility for Application Profiles**. In A. Jatowt, A. Maeda, & S. Y. Syn (Eds.), *Digital Libraries at the Crossroads of Digital Information for the Future* (pp. 116–122). Lecture Notes in Computer Science. Springer International Publishing. <https://doi.org/10.1007/978-3-030-34058-2_12>

2. Thalhath, N., Nagamori, M., Sakaguchi, T., & Sugimoto, S. (2019). **Yet Another Metadata Application Profile (YAMA): Authoring, Versioning and Publishing of Application Profiles**. *International Conference on Dublin Core and Metadata Applications*, 114–125. <https://doi.org/10.23106/dcmi.952141854>

3. Thalhath, N., Nagamori, M., Sakaguchi, T., & Sugimoto, S. (2020). **Metadata Application Profile Provenance with Extensible Authoring Format and PAV Ontology**. In X. Wang, F. A. Lisi, G. Xiao, & E. Botoeva (Eds.), *Semantic Technology* (pp. 353–368). Lecture Notes in Computer Science. Springer International Publishing. <https://doi.org/10.1007/978-3-030-41407-8_23>

4. Thalhath, N., Nagamori, M., & Sakaguchi, T. (2022). **YAMAML: An Application Profile Based Lightweight RDF Mapping Language**. In Y.-H. Tseng, M. Katsurai, & H. N. Nguyen (Eds.), *From Born-Physical to Born-Virtual: Augmenting Intelligence in Digital Libraries* (pp. 412–420). Lecture Notes in Computer Science. Springer International Publishing. <https://doi.org/10.1007/978-3-031-21756-2_32>

5. Thalhath, N., Nagamori, M., & Sakaguchi, T. (2025). **Metadata Application Profile as a Mechanism for Semantic Interoperability in FAIR and Open Data Publishing**. *Data and Information Management*, *9*(1), 100068. <https://doi.org/10.1016/j.dim.2024.100068>

## License

This specification is licensed under [Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)](https://creativecommons.org/licenses/by-sa/4.0/).
