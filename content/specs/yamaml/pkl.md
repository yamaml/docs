---
title: Authoring in PKL
weight: 2
toc: true
---

**PKL** (pronounced *Pickle*) is [Apple's configuration language](https://pkl-lang.org) with strong typing, inheritance, and rendering to JSON / YAML / Plist / Java properties. Because PKL can render to YAML, it works as a **type-safe authoring layer on top of YAMAML**: you write your profile in PKL, render it to YAMAML, and feed it to the existing `yama` CLI.

This page describes the `yama.pkl` schema package and shows how every YAMAML feature maps to PKL syntax. The underlying semantics are identical — PKL is simply an alternative surface syntax.

{{< callout type="info" >}}
  This is an **authoring layer**, not a new format. The canonical source of truth remains the [YAMAML specification](spec). PKL files render to YAMAML before being processed.
{{< /callout >}}

## 1. Why PKL?

YAMAML (YAML) is excellent for small profiles and quick authoring. PKL shines when profiles grow:

| Concern | YAMAML | PKL |
|---|---|---|
| Authoring-time validation | ✗ Runtime only | ✓ Type-checked in editor |
| Reusable base profiles | Anchors/aliases (limited) | `amends` / `extends` across files |
| Editor autocomplete | Generic YAML | Full IntelliSense from schema |
| Computed values | ✗ | ✓ Functions, interpolation |
| Modular imports | ✗ Single-file only | ✓ HTTPS / package imports |
| Renders to YAMAML | — | ✓ Via `pkl eval -f yaml` |

A typo like `tyep: literal` fails at generation time in YAMAML; the PKL compiler catches it the moment you save the file.

## 2. The Workflow

```
profile.pkl  ──pkl eval──→  profile.yaml  ──yama package──→  RDF, SHACL, ShEx, DSP, ...
```

You never lose access to any YAMA feature — everything the `yama` CLI accepts in YAMAML is expressible in PKL, and the rendered YAMAML is indistinguishable from hand-authored YAMAML.

```sh
# One-shot: render PKL and generate a full profile package
pkl eval -f yaml profile.pkl | yama package -i - -o dist/
```

## 3. Installing PKL

```sh
# macOS / Homebrew
brew install pkl

# Linux
curl -L -o pkl https://github.com/apple/pkl/releases/latest/download/pkl-linux-amd64
chmod +x pkl && sudo mv pkl /usr/local/bin/

# Verify
pkl --version
```

See the [PKL installation guide](https://pkl-lang.org/main/current/pkl-cli/index.html) for Windows and Docker options.

## 4. The `yama.pkl` Schema Package

The schema is published as a PKL package at **[github.com/yamaml/yama-pkl](https://github.com/yamaml/yama-pkl)** and resolvable via the `pkg.pkl-lang.org` redirect service. To use it, `amends` the `Profile` module:

```pkl {filename="profile.pkl"}
amends "package://pkg.pkl-lang.org/github.com/yamaml/yama-pkl/yama@0.1.0#/Profile.pkl"

base = "http://example.org/people/"

namespaces {
  ["foaf"] = "http://xmlns.com/foaf/0.1/"
  ["xsd"]  = "http://www.w3.org/2001/XMLSchema#"
}

descriptions {
  ["person"] = new Description {
    a = "foaf:Person"
    statements {
      ["name"] = new LiteralStatement {
        property = "foaf:name"
        min = 1
        max = 1
        datatype = "xsd:string"
      }
    }
  }
}
```

The URL format decomposes as:

| Segment | Meaning |
|---|---|
| `package://pkg.pkl-lang.org` | PKL package redirect service |
| `github.com/yamaml/yama-pkl` | GitHub repository hosting the releases |
| `yama@0.1.0` | `<packageName>@<version>` |
| `#/Profile.pkl` | Asset path inside the package zip |

Behind the scenes, `pkg.pkl-lang.org` rewrites this URL to the GitHub release download so PKL can fetch the package zip — there is no additional hosting infrastructure to run.

### Classes provided

The `Profile` module defines the following typed classes, each with inline documentation accessible via your editor's hover/go-to-definition:

| Class | Purpose |
|---|---|
| `Profile` *(open module)* | Top-level document — `amends` this in your own file |
| `Description` | A class of resources — shape, template, or record type |
| `Statement` *(abstract)* | Base class for property constraints — use a concrete subclass |
| `LiteralStatement` | Literal values (text, number, date, picklist, pattern, facets) |
| `IriStatement` | URI/IRI references, linked descriptions, vocabulary schemes |
| `BnodeStatement` | Inline blank-node structured values (`description` required) |
| `Facets` | XSD numeric and string facets (`MinInclusive`, `Length`, …) |
| `IdMapping` | Identifier configuration for data-mapped descriptions |
| `DataMapping` | Data source mapping for RDF generation |
| `Defaults` | Inherited defaults for mapping |

The full schema source lives at [`Profile.pkl`](https://github.com/yamaml/yama-pkl/blob/main/Profile.pkl) in the yama-pkl repository, where every class and field is documented with its purpose and its mapping to the YAMAML specification.

## 5. Document Structure

A YAMA PKL file `amends` the `Profile` module and fills in its fields. This is the PKL equivalent of the YAMAML top-level mapping described in [Section 2 of the YAMAML specification](spec#2-document-structure).

```pkl {filename="profile.pkl"}
amends "package://pkg.pkl-lang.org/github.com/yamaml/yama-pkl/yama@0.1.0#/Profile.pkl"

base = "http://example.org/people/"

namespaces {
  ["foaf"]    = "http://xmlns.com/foaf/0.1/"
  ["schema"]  = "http://schema.org/"
  ["xsd"]     = "http://www.w3.org/2001/XMLSchema#"
  ["dcterms"] = "http://purl.org/dc/terms/"
}

descriptions {
  ["person"] = new Description {
    a = "foaf:Person"
    label = "Person"
    // ...
  }
}
```

Rendering this file with `pkl eval -f yaml profile.pkl` produces exactly the YAMAML document you would hand-author.

## 6. Namespaces

Namespaces are a `Mapping<String, String>`. PKL's `["key"] = value` syntax declares dynamic keys:

```pkl
namespaces {
  ["foaf"]   = "http://xmlns.com/foaf/0.1/"
  ["schema"] = "http://schema.org/"
  ["xsd"]    = "http://www.w3.org/2001/XMLSchema#"
}
```

You can also build namespaces compositionally by amending a base module that defines common prefixes.

## 7. Descriptions

A description is a typed object. Its fields mirror the YAMAML description keys from [Section 3](spec#3-descriptions):

```pkl
descriptions {
  ["book"] = new Description {
    a = "schema:Book"
    label = "Book"
    note = "A published book"
    statements {
      ["title"] = new LiteralStatement {
        property = "dcterms:title"
        min = 1
        max = 1
        datatype = "xsd:string"
      }
    }
  }
}
```

### Closed shapes

The `closed` field maps directly to SHACL's `sh:closed`:

```pkl
["book"] = new Description {
  a = "schema:Book"
  closed = true
  statements { /* ... */ }
}
```

### Identifier mapping

When a description participates in RDF generation, add an `id` mapping:

```pkl
["character"] = new Description {
  a = "foaf:Person"
  id = new IdMapping {
    prefix = "tbbt"
    mapping = new DataMapping {
      source = "characters.csv"
      `type` = "csv"
      path = "ID"
    }
  }
  statements { /* ... */ }
}
```

### Description references

IRI and BNODE statements reference other descriptions by key:

```pkl
["person"] = new Description {
  a = "foaf:Person"
  statements {
    ["address"] = new BnodeStatement {
      property = "schema:address"
      description = "postalAddress"   // inline blank node
    }
    ["knows"] = new IriStatement {
      property = "foaf:knows"
      description = "person"          // IRI link, self-reference
    }
  }
}

["postalAddress"] = new Description {
  a = "schema:PostalAddress"
  statements {
    ["street"] = new LiteralStatement {
      property = "schema:streetAddress"
      datatype = "xsd:string"
    }
  }
}
```

### 7.1 Multi-shape references (disjunctions)

Both `IriStatement` and `BnodeStatement` accept `description` as either a single string or a list of strings. A list expresses that the value may conform to any one of the named descriptions:

```pkl
["creator"] = new IriStatement {
  property = "dcterms:creator"
  description = new Listing<String> { "Person"; "Organization" }
}
```

This renders to YAMAML as:

```yaml
creator:
  property: dcterms:creator
  type: IRI
  description:
    - Person
    - Organization
```

Downstream generators map the list to the appropriate disjunction idiom of each target format: `sh:or` in SHACL, `(@<A> OR @<B>)` in ShEx, `owl:unionOf` in OWL-DSP, space-separated `valueShape` in DCTAP (following [DCMI SRAP](https://github.com/dcmi/dc-srap)), and space-separated `#A #B` in SimpleDSP (yama-cli extension).

## 8. Statements

Statement fields map 1-to-1 with those in [Section 4 of the YAMAML specification](spec#4-statements). The key PKL-specific choice is **which statement subclass to instantiate**:

| Subclass | Used for | Maps to YAMAML |
|---|---|---|
| `LiteralStatement` | Text values, numbers, dates, picklists | `type: literal` (default) |
| `IriStatement` | URI references, linked resources | `type: IRI` |
| `BnodeStatement` | Inline blank nodes (nested structures) | `type: BNODE` |

The `` `type` `` field (escaped with backticks because `type` is a PKL keyword) is set automatically by the subclass.

### 8.1 Cardinality

```pkl
["title"] = new LiteralStatement {
  property = "dcterms:title"
  min = 1        // required
  max = 1        // single-valued
}

["author"] = new LiteralStatement {
  property = "dcterms:creator"
  min = 1        // at least one required
  // max omitted = unbounded
}
```

For descriptive cardinality that doesn't fit numeric min/max, use `cardinalityNote`:

```pkl
["notes"] = new LiteralStatement {
  property = "dcterms:description"
  cardinalityNote = "recommended"
  datatype = "xsd:string"
}
```

### 8.2 Datatype, values, and pattern

```pkl
// Datatype
["price"] = new LiteralStatement {
  property = "schema:price"
  datatype = "xsd:decimal"
}

// Enumerated values (picklist)
["category"] = new LiteralStatement {
  property = "schema:category"
  values {
    "electronics"
    "clothing"
    "books"
    "food"
  }
}

// Regex pattern
["isbn"] = new LiteralStatement {
  property = "schema:isbn"
  datatype = "xsd:string"
  pattern = #"^\d{4}-\d{3}[\dX]$"#
}
```

{{< callout type="info" >}}
  PKL's custom-delimited strings (`#"..."#`) make regexes readable without backslash doubling — a quiet ergonomic win over YAML.
{{< /callout >}}

### 8.3 Facets

```pkl
["age"] = new LiteralStatement {
  property = "foaf:age"
  datatype = "xsd:integer"
  facets = new Facets {
    MinInclusive = 0
    MaxInclusive = 150
  }
}

["isbn"] = new LiteralStatement {
  property = "schema:isbn"
  datatype = "xsd:string"
  facets = new Facets {
    Length = 13
  }
}
```

All facets defined in [Section 4.4 of the YAMAML specification](spec#44-facets) are available as typed fields on the `Facets` class.

### 8.4 Vocabulary scheme

```pkl
["subject"] = new IriStatement {
  property = "dcterms:subject"
  inScheme = "skos:"              // single namespace
}

["classification"] = new IriStatement {
  property = "dcterms:subject"
  inScheme = new Listing { "ndlsh:"; "lcsh:" }   // multiple
}
```

## 9. Data Mapping

The `DataMapping` class covers every property from [Section 5](spec#5-data-mapping):

```pkl
["email"] = new LiteralStatement {
  property = "foaf:mbox"
  mapping = new DataMapping {
    path = "email"
    prepend = "mailto:"
  }
}

["friends"] = new IriStatement {
  property = "foaf:knows"
  description = "person"
  mapping = new DataMapping {
    path = "friends"
    separator = ","
  }
}
```

Defaults work through the top-level `defaults` field:

```pkl
defaults = new Defaults {
  mapping = new DataMapping {
    source = "people.csv"
    `type` = "csv"
  }
}
```

## 10. Complete Examples

The three following examples correspond 1-to-1 with those in the [YAMAML Examples](../examples/). Each PKL file renders to the YAMAML shown in those pages.

### 10.1 Metadata Profile (No Data Mapping)

```pkl {filename="manga-catalog.pkl"}
amends "package://pkg.pkl-lang.org/github.com/yamaml/yama-pkl/yama@0.1.0#/Profile.pkl"

base = "http://example.org/manga/"

namespaces {
  ["schema"]  = "http://schema.org/"
  ["xsd"]     = "http://www.w3.org/2001/XMLSchema#"
  ["dcterms"] = "http://purl.org/dc/terms/"
  ["foaf"]    = "http://xmlns.com/foaf/0.1/"
}

descriptions {
  ["series"] = new Description {
    a = "schema:ComicSeries"
    label = "Manga Series"
    note = "A manga series or title"
    statements {
      ["title"] = new LiteralStatement {
        label = "Title"
        property = "schema:name"
        min = 1; max = 1
        datatype = "xsd:string"
      }
      ["genre"] = new LiteralStatement {
        label = "Genre"
        property = "schema:genre"
        min = 1
        datatype = "xsd:string"
      }
      ["author"] = new IriStatement {
        label = "Author"
        property = "schema:author"
        min = 1
        description = "creator"
      }
      ["volume"] = new IriStatement {
        label = "Volumes"
        property = "schema:hasPart"
        min = 1
        description = "volume"
      }
    }
  }

  ["volume"] = new Description {
    a = "schema:Book"
    label = "Volume"
    note = "A single volume of a manga series"
    statements {
      ["volumeNumber"] = new LiteralStatement {
        label = "Volume Number"
        property = "schema:volumeNumber"
        min = 1; max = 1
        datatype = "xsd:integer"
      }
      ["datePublished"] = new LiteralStatement {
        label = "Date Published"
        property = "schema:datePublished"
        min = 1; max = 1
        datatype = "xsd:date"
      }
    }
  }

  ["creator"] = new Description {
    a = "foaf:Person"
    label = "Creator"
    note = "A manga author or artist"
    statements {
      ["name"] = new LiteralStatement {
        label = "Name"
        property = "schema:name"
        min = 1; max = 1
        datatype = "xsd:string"
      }
      ["birthDate"] = new LiteralStatement {
        label = "Birth Date"
        property = "schema:birthDate"
        min = 0; max = 1
        datatype = "xsd:date"
      }
    }
  }
}
```

Render and build:

```sh
pkl eval -f yaml manga-catalog.pkl | yama package -i - -o dist/
```

### 10.2 Data-Mapped Profile (RDF Generation)

```pkl {filename="people.pkl"}
amends "package://pkg.pkl-lang.org/github.com/yamaml/yama-pkl/yama@0.1.0#/Profile.pkl"

base = "http://example.org/people/"

namespaces {
  ["foaf"]   = "http://xmlns.com/foaf/0.1/"
  ["schema"] = "http://schema.org/"
  ["xsd"]    = "http://www.w3.org/2001/XMLSchema#"
  ["rdfs"]   = "http://www.w3.org/2000/01/rdf-schema#"
}

defaults = new Defaults {
  mapping = new DataMapping {
    source = "people.csv"
    `type` = "csv"
  }
}

descriptions {
  ["person"] = new Description {
    a = "foaf:Person"
    label = "Person"
    note = "A person record"
    id = new IdMapping {
      mapping = new DataMapping { path = "ID" }
    }
    statements {
      ["name"] = new LiteralStatement {
        label = "Name"
        property = "foaf:name"
        min = 1; max = 1
        datatype = "xsd:string"
        mapping = new DataMapping { path = "name" }
      }
      ["email"] = new IriStatement {
        label = "Email"
        property = "foaf:mbox"
        min = 0
        mapping = new DataMapping {
          path = "email"
          prepend = "mailto:"
        }
      }
      ["knows"] = new IriStatement {
        label = "Knows"
        property = "foaf:knows"
        min = 0
        description = "person"
        mapping = new DataMapping {
          path = "friends"
          separator = ","
        }
      }
      ["address"] = new BnodeStatement {
        label = "Address"
        property = "schema:address"
        min = 0; max = 1
        description = "postalAddress"
      }
    }
  }

  ["postalAddress"] = new Description {
    a = "schema:PostalAddress"
    label = "Address"
    statements {
      ["street"] = new LiteralStatement {
        label = "Street"
        property = "schema:streetAddress"
        datatype = "xsd:string"
        mapping = new DataMapping { path = "street" }
      }
      ["city"] = new LiteralStatement {
        label = "City"
        property = "schema:addressLocality"
        datatype = "xsd:string"
        mapping = new DataMapping { path = "city" }
      }
      ["postalCode"] = new LiteralStatement {
        label = "Postal Code"
        property = "schema:postalCode"
        datatype = "xsd:string"
        mapping = new DataMapping { path = "zip" }
      }
    }
  }
}
```

### 10.3 Rich Constraints

```pkl {filename="products.pkl"}
amends "package://pkg.pkl-lang.org/github.com/yamaml/yama-pkl/yama@0.1.0#/Profile.pkl"

base = "http://example.org/products/"

namespaces {
  ["schema"] = "http://schema.org/"
  ["xsd"]    = "http://www.w3.org/2001/XMLSchema#"
}

descriptions {
  ["product"] = new Description {
    a = "schema:Product"
    label = "Product"
    closed = true
    statements {
      ["name"] = new LiteralStatement {
        label = "Name"
        property = "schema:name"
        min = 1; max = 1
        datatype = "xsd:string"
      }
      ["price"] = new LiteralStatement {
        label = "Price"
        property = "schema:price"
        min = 1; max = 1
        datatype = "xsd:decimal"
        facets = new Facets {
          MinInclusive = 0
          MaxInclusive = 999999.99
        }
      }
      ["sku"] = new LiteralStatement {
        label = "SKU"
        property = "schema:sku"
        min = 1; max = 1
        datatype = "xsd:string"
        pattern = #"^[A-Z]{3}-\d{6}$"#
        facets = new Facets { Length = 10 }
      }
      ["category"] = new LiteralStatement {
        label = "Category"
        property = "schema:category"
        min = 1; max = 1
        values {
          "electronics"
          "clothing"
          "books"
          "food"
        }
      }
      ["manufacturer"] = new IriStatement {
        label = "Manufacturer"
        property = "schema:manufacturer"
        min = 0; max = 1
        description = "organization"
      }
    }
  }

  ["organization"] = new Description {
    a = "schema:Organization"
    label = "Organization"
    statements {
      ["name"] = new LiteralStatement {
        label = "Name"
        property = "schema:name"
        min = 1; max = 1
        datatype = "xsd:string"
      }
      ["url"] = new IriStatement {
        label = "Website"
        property = "schema:url"
        min = 0; max = 1
      }
    }
  }
}
```

## 11. Composition: Reusable Base Profiles

One of PKL's biggest advantages over YAML is that you can `amends` a shared base profile and override only what you need. This is invaluable for organizations maintaining many related profiles:

```pkl {filename="base-person.pkl"}
amends "package://pkg.pkl-lang.org/github.com/yamaml/yama-pkl/yama@0.1.0#/Profile.pkl"

namespaces {
  ["foaf"]    = "http://xmlns.com/foaf/0.1/"
  ["schema"]  = "http://schema.org/"
  ["xsd"]     = "http://www.w3.org/2001/XMLSchema#"
}

descriptions {
  ["person"] = new Description {
    a = "foaf:Person"
    statements {
      ["name"] = new LiteralStatement {
        property = "foaf:name"
        min = 1; max = 1
        datatype = "xsd:string"
      }
      ["email"] = new LiteralStatement {
        property = "foaf:mbox"
        datatype = "xsd:anyURI"
      }
    }
  }
}
```

```pkl {filename="customer.pkl"}
amends "base-person.pkl"

base = "http://example.org/customers/"

// Extend the person description with customer-specific fields
descriptions {
  ["person"] {
    a = "schema:Customer"    // override the class
    statements {
      ["loyaltyId"] = new LiteralStatement {
        property = "schema:identifier"
        datatype = "xsd:string"
      }
    }
  }
}
```

The `customer.pkl` file inherits every namespace, description, and statement from `base-person.pkl`, then overrides and extends only what matters. Hand-maintaining this in YAML would require either copy-paste or a custom pre-processor.

## 12. When to use PKL vs YAMAML

|  | Choose YAMAML (YAML) when... | Choose PKL when... |
|---|---|---|
| **Profile size** | A few descriptions, < 50 statements | Many descriptions, or large statement counts |
| **Sharing** | Single-file profile | Shared base profiles across teams |
| **Validation** | Willing to catch errors at `yama validate` | Want type-checking in the editor |
| **Tooling** | Any text editor, no extra install | Willing to install PKL + editor plugin |
| **Learning curve** | Already know YAML | Willing to learn PKL's type system |
| **Canonical file** | Yes — the `.yaml` is what you commit | The `.pkl` is the source; render on CI |

Both formats produce the same outputs. You can switch between them at any time — render your PKL to YAMAML once and commit the result, or go the other direction by translating a YAMAML file into PKL.

## 13. See Also

- [YAMAML Specification](spec) — the normative language definition, which PKL mirrors 1-to-1
- [PKL Language Reference](https://pkl-lang.org/main/current/language-reference/index.html) — PKL syntax, classes, and rendering
- [PKL CLI Reference](https://pkl-lang.org/main/current/pkl-cli/index.html) — `pkl eval`, `pkl repl`, and output formats
- [yama-pkl repository](https://github.com/yamaml/yama-pkl) — schema source, release workflow, and examples
- [yama-pkl API docs](https://yamaml.github.io/yama-pkl/) — generated API reference for the PKL schema
