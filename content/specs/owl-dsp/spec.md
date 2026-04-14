---
title: OWL-DSP Specification
weight: 1
toc: true
description: "OWL-DSP is an OWL ontology for describing Description Set Profiles. English specification based on the original Japanese document published by the Japanese Ministry of Internal Affairs and Communications (総務省)."
---

{{< callout type="info" >}}
This English specification is based on the official Japanese specification document
[*Guidelines for Metadata Information Sharing*, Chapter 6: Metadata Schema Definition Language](https://www.soumu.go.jp/main_content/000132512.pdf)
(『メタデータ情報共有のためのガイドライン 第6章 メタデータ・スキーマ定義言語』), sections 6.1 and 6.3,
published (circa 2011) on the website of the **Ministry of Internal Affairs and Communications of Japan (総務省)**
as part of the *Metadata Information Infrastructure Construction Project* (メタデータ情報基盤構築事業), whose work was subsequently carried on by the [Metadata Information Infrastructure Initiative (MI3)](https://web.archive.org/web/20200209202846/http://mi3.or.jp/) (メタデータ基盤協議会, Japan).
The original Japanese text is preserved verbatim in the [OWL-DSP 仕様書（日本語原文）](../spec-original-ja) page.
{{< /callout >}}

---

## Overview

OWL-DSP (Description Set Profile Definition Language) is a meta-language for expressing metadata description rules using OWL (Web Ontology Language). It encodes:

- **Record description rules** (Description Templates) as OWL classes
- **Item description rules** (Statement Templates) as OWL class restrictions

By expressing metadata as instances of Description Template classes, OWL reasoners can perform consistency validation on metadata records.

**Ontology namespace:** `http://purl.org/metainfo/terms/dsp#`

**Version:** 0.30 (created 2010-12-20, modified 2011-02-13)

---

## Classes

### `dsp:DescriptionTemplate`

A metaclass representing a record description rule.

```
dsp:DescriptionTemplate rdfs:subClassOf owl:Class .
```

A Description Template D is expressed as an OWL class (e.g. `ex:CD`). Metadata written according to D (e.g. `ex:MD`) becomes an instance of `ex:CD`. That is:

```turtle
ex:CD a owl:Class .    # or: ex:CD a dsp:DescriptionTemplate .
ex:MD a ex:CD .        # metadata instance conforming to the template
```

To make it explicit that `ex:CD` is a Description Template rather than a plain OWL class, it may be expressed as an instance of `dsp:DescriptionTemplate`.

Statement Templates are linked to Description Templates via `rdfs:subClassOf` relationships.

### `dsp:StatementTemplate`

A metaclass representing an item description rule (property constraint).

```
dsp:StatementTemplate rdfs:subClassOf owl:Restriction .
```

A Statement Template S is expressed as an OWL class restriction (e.g. `ex:RS`). The Description Template D that contains S has `ex:CD rdfs:subClassOf ex:RS`. If D1 has Statement Templates S1, S2, S3:

```turtle
ex:CD1 rdfs:subClassOf ex:RS1, ex:RS2, ex:RS3 .
```

Metadata conforming to D1 must satisfy all constraints S1, S2, S3 (the intersection of the restrictions).

Each Statement Template must define exactly one target property (`owl:onProperty`).

---

## Properties

### `dsp:valueURIOccurrence` (DatatypeProperty)

Indicates whether records may be blank nodes.

| Value | Meaning |
|-------|---------|
| `"mandatory"` | URI required (records must have a URI) |
| `"optional"` | Blank nodes permitted (default if property absent) |
| `"disallowed"` | Records must always be blank nodes |

- **Domain:** `dsp:DescriptionTemplate`
- **Range:** `{"mandatory", "optional", "disallowed"}`

### `dsp:inScheme` (ObjectProperty)

Indicates that members of the class are composed of concepts from a vocabulary (thesaurus, etc.) identified by the object.

For example, to constrain `dc:subject` values to NDLSH terms:

```turtle
[dsp:StatementTemplate ;
   owl:onProperty dc:subject ;
   owl:onClass [dsp:inScheme ndlsh:]] .
```

For multiple vocabularies, use `owl:unionOf`:

```turtle
owl:onClass [owl:unionOf(
   [dsp:inScheme ndlsh:]
   [dsp:inScheme bsh:]
)] .
```

The inference rule is: `{ex:CR dsp:inScheme ndlsh:}` implies `{ex:CR owl:onProperty skos:inScheme; owl:allValuesFrom ndlsh:}`.

Comparable to DCMI-DSP's `vocabularyEncodingScheme`.

- **Domain:** `rdfs:Class`
- **Range:** `skos:ConceptScheme`

### `dsp:resourceClass` (ObjectProperty)

Specifies the RDF class that metadata instances described by this Description Template will be members of. Equivalent to DCMI-DSP's `resourceClass`.

- **Domain:** `dsp:DescriptionTemplate`
- **Range:** `rdfs:Class`

### `dsp:cardinalityNote` (DatatypeProperty)

Records non-numeric occurrence constraints such as "recommended" (推奨) or "mandatory if present" (あれば必須). When this property is present, `owl:minCardinality` is interpreted as 1.

- **Domain:** `dsp:StatementTemplate`

### `dsp:langTagOccurrence` (DatatypeProperty)

For Statement Templates whose value constraint is a plain literal, indicates whether a language tag is mandatory, optional, or disallowed. Using this property on a non-plain-literal Statement Template is an error.

| Value | Meaning |
|-------|---------|
| `"mandatory"` | Language tag required |
| `"optional"` | Language tag optional |
| `"disallowed"` | Language tag must not be present |

- **Domain:** `dsp:StatementTemplate`
- **Range:** `{"mandatory", "optional", "disallowed"}`

### `dsp:perLangMaxCardinality` (DatatypeProperty)

Constrains the maximum number of occurrences per language tag. Useful when property values include readings (pronunciations) expressed via language tags — setting this to 1 guarantees a 1:1 correspondence between value strings and readings. Using this on a non-plain-literal Statement Template is an error.

- **Domain:** `dsp:StatementTemplate`
- **Range:** `xsd:nonNegativeInteger`

### `dsp:propertyMapping` (ObjectProperty)

Associates a Statement Template's property P with a general-purpose upper property Q for "dumb-down" (simplification) purposes. Normally `P rdfs:subPropertyOf Qp` would be used, but this property is needed when:

- A different property mapping than `subPropertyOf` is desired
- P is from an external vocabulary and cannot have `subPropertyOf` added

The implication is: `{[a dsp:StatementTemplate; owl:onProperty any:P; dsp:propertyMapping ex:Q]}` implies `{any:P rdfs:subPropertyOf ex:Q}`.

- **Domain:** `dsp:StatementTemplate`
- **Range:** `rdf:Property`

---

## Class and Property Summary

| URI | Type | Label | Domain | Range |
|-----|------|-------|--------|-------|
| `dsp:DescriptionTemplate` | `owl:Class` | Description Template | — | — |
| `dsp:StatementTemplate` | `owl:Class` | Statement Template | — | — |
| `dsp:valueURIOccurrence` | `owl:DatatypeProperty` | Value URI Occurrence | `dsp:DescriptionTemplate` | `{"mandatory","optional","disallowed"}` |
| `dsp:inScheme` | `owl:ObjectProperty` | In Scheme | `rdfs:Class` | `skos:ConceptScheme` |
| `dsp:resourceClass` | `owl:ObjectProperty` | Resource Class | `dsp:DescriptionTemplate` | `rdfs:Class` |
| `dsp:cardinalityNote` | `owl:DatatypeProperty` | Cardinality Note | `dsp:StatementTemplate` | — |
| `dsp:langTagOccurrence` | `owl:DatatypeProperty` | Language Tag Occurrence | `dsp:StatementTemplate` | `{"mandatory","optional","disallowed"}` |
| `dsp:perLangMaxCardinality` | `owl:DatatypeProperty` | Per Language Max Cardinality | `dsp:StatementTemplate` | `xsd:nonNegativeInteger` |
| `dsp:propertyMapping` | `owl:ObjectProperty` | Property Mapping | `dsp:StatementTemplate` | `rdf:Property` |

---

## Worked Example (Section 6.3)

The following shows the [SimpleDSP](/specs/simpledsp/spec/) example from section 6.2.5 converted to OWL-DSP with registry metadata. This demonstrates how a tabular SimpleDSP file maps to the OWL-DSP ontology.

### SimpleDSP Input

```
[@NS]
dcndl	http://ndl.go.jp/dcndl/terms/
ndlsh	http://id.ndl.go.jp/auth/ndlsh/
bsh	http://id.ndl.go.jp/auth/bsh/
ndlbooks	http://iss.ndl.go.jp/books/
@base	http://ndl.go.jp/dcndl/dsp/biblio

[MAIN]
#項目規則名	プロパティ	最小	最大	値タイプ	値制約	説明
書誌ID	foaf:Document	1	1	ID	ndlbooks:	文書のID
タイトル	dcterms:title	1	1	構造化	#構造化タイトル	文書の表題
著者	dcterms:creator	0	1	構造化	foaf:Agent	文書の作者
発行日	dcterms:issued	1	1	文字列	xsd:date	文書の発行日
主題	dcterms:subject	0	-	参照値	ndlsh: bsh:	文書の主題

[構造化タイトル]
#項目規則名	プロパティ	最小	最大	値タイプ	値制約	説明
リテラル値	xl:literalForm	1	1	文字列		タイトル自身
読み	dcndl:transcription	0	1	文字列		タイトルの読み
```

### OWL-DSP Output

```turtle
@prefix bsh: <http://id.ndl.go.jp/auth/bsh/>.
@prefix ndlbooks: <http://iss.ndl.go.jp/books/>.
@prefix owl: <http://www.w3.org/2002/07/owl#>.
@prefix dsp: <http://purl.org/metainfo/terms/dsp#>.
@prefix xsd: <http://www.w3.org/2001/XMLSchema#>.
@prefix dcndl: <http://ndl.go.jp/dcndl/terms/>.
@prefix reg: <http://purl.org/metainfo/terms/registry#>.
@prefix foaf: <http://xmlns.com/foaf/0.1/>.
@prefix xl: <http://www.w3.org/2008/05/skos-xl#>.
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>.
@prefix ndlsh: <http://id.ndl.go.jp/auth/ndlsh/>.
@prefix dcterms: <http://purl.org/dc/terms/>.
@base <http://ndl.go.jp/dcndl/dsp/biblio>.

<> a owl:Ontology ;
   rdfs:label "National Diet Library Bibliographic Description" ;
   reg:created "2011-01-15" ;
   reg:creator ex:aRegisteredUserId ;
   reg:version "Version 1.1" .
```

#### MAIN Description Template

The `[MAIN]` block becomes a `dsp:DescriptionTemplate` with its Statement Templates as subclasses:

```turtle
<#MAIN> a dsp:DescriptionTemplate ;
   dsp:valueURIOccurrence "mandatory" ;       # ID row → URI is required
   dsp:resourceClass foaf:Document ;           # ID row Property column → class
   reg:idField "書誌ID" ;                      # ID row Name column
   reg:resourceNsURI ndlbooks: ;               # ID row Constraint column
   rdfs:subClassOf <#MAIN-タイトル>, <#MAIN-著者>, <#MAIN-発行日>, <#MAIN-主題> .
```

#### Statement Templates

Each non-ID row becomes a `dsp:StatementTemplate`:

```turtle
# タイトル → structured, references #構造化タイトル block
<#MAIN-タイトル> a dsp:StatementTemplate ;
   rdfs:label "タイトル" ;
   owl:onProperty dcterms:title ;
   owl:qualifiedCardinality 1 ;          # min=1, max=1
   owl:onClass <#構造化タイトル> ;        # structured → owl:onClass
   rdfs:comment "文書の表題" .

# 著者 → structured, class constraint foaf:Agent
<#MAIN-著者> a dsp:StatementTemplate ;
   rdfs:label "著者" ;
   owl:onProperty dcterms:creator ;
   dsp:cardinalityNote "あれば必須" ;     # min=0 with keyword → cardinalityNote
   owl:onClass foaf:Agent ;               # structured → owl:onClass
   rdfs:comment "文書の作者" .

# 発行日 → literal with xsd:date datatype
<#MAIN-発行日> a dsp:StatementTemplate ;
   rdfs:label "発行日" ;
   owl:onProperty dcterms:issued ;
   dsp:cardinalityNote "あれば必須" ;
   owl:maxQualifiedCardinality 1 ;
   owl:onDataRange xsd:date ;             # literal → owl:onDataRange
   rdfs:comment "文書の発行日" .

# 主題 → reference with vocabulary schemes ndlsh: and bsh:
<#MAIN-主題> a dsp:StatementTemplate ;
   rdfs:label "主題" ;
   owl:onProperty dcterms:subject ;
   owl:onClass [owl:unionOf(              # reference + multiple vocabs → unionOf
      [dsp:inScheme ndlsh:]               #   → dsp:inScheme
      [dsp:inScheme bsh:]
   )] ;
   rdfs:comment "文書の主題" .
```

#### Nested Description Template

The `[構造化タイトル]` block (referenced as `#構造化タイトル`) becomes its own `dsp:DescriptionTemplate`:

```turtle
<#構造化タイトル> a dsp:DescriptionTemplate ;
   rdfs:subClassOf <#構造化タイトル-リテラル値>, <#構造化タイトル-読み> .

<#構造化タイトル-リテラル値> a dsp:StatementTemplate ;
   rdfs:label "リテラル値" ;
   owl:onProperty xl:literalForm ;
   owl:qualifiedCardinality 1 ;
   owl:onDataRange rdfs:Literal ;
   rdfs:comment "タイトル自身" .

<#構造化タイトル-読み> a dsp:StatementTemplate ;
   rdfs:label "読み" ;
   owl:onProperty dcndl:transcription ;
   owl:maxQualifiedCardinality 1 ;
   owl:onDataRange rdfs:Literal ;
   rdfs:comment "タイトルの読み" .
```

---

## SimpleDSP → OWL-DSP Mapping Summary

| SimpleDSP Element | OWL-DSP Representation |
|---|---|
| Block `[ID]` | `<#ID> a dsp:DescriptionTemplate` |
| Row (non-ID) | `<#block-name> a dsp:StatementTemplate` |
| Property column | `owl:onProperty` |
| Min = *n* | `owl:minQualifiedCardinality n` |
| Max = *n* | `owl:maxQualifiedCardinality n` |
| Min = *n*, Max = *n* (same) | `owl:qualifiedCardinality n` |
| Min keyword (推奨 etc.) | `dsp:cardinalityNote "keyword"` |
| ValueType = `literal` | Constraint → `owl:onDataRange` |
| ValueType = `structured` + `#ref` | Constraint → `owl:onClass <#ref>` |
| ValueType = `structured` + class | Constraint → `owl:onClass className` |
| ValueType = `reference` + vocab | Constraint → `owl:onClass [dsp:inScheme vocab:]` |
| ValueType = `ID` (Property col) | → `dsp:resourceClass` |
| ValueType = `ID` (Constraint col) | → `reg:resourceNsURI` / `dsp:valueURIOccurrence "mandatory"` |
| Comment column | `rdfs:comment` |
| Name column | `rdfs:label` |

## References

- Kanzaki, M. *OWL-DSP — Description Set Profile Definition Language*: <https://www.kanzaki.com/ns/dsp#>
- OWL-DSP Ontology namespace: `http://purl.org/metainfo/terms/dsp#`
- [*Guidelines for Metadata Information Sharing* (メタデータ情報共有のためのガイドライン), Metadata Information Infrastructure Initiative (MI3 — メタデータ基盤協議会)](https://www.soumu.go.jp/main_content/000132512.pdf)
- NDL Current Awareness-R (2011-09-21): *メタデータ情報基盤構築事業で構築されたメタデータレジストリ "Meta Bridge"* — <https://current.ndl.go.jp/car/19135>
