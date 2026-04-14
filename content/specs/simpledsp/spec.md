---
title: SimpleDSP Specification
weight: 1
toc: true
description: "SimpleDSP is a tab-separated text format for defining metadata Description Templates. English specification based on the original Japanese document published by the Japanese Ministry of Internal Affairs and Communications (総務省)."
---

{{< callout type="info" >}}
This English specification is based on the official Japanese specification document
[*Guidelines for Metadata Information Sharing*, Chapter 6: Metadata Schema Definition Language](https://www.soumu.go.jp/main_content/000132512.pdf)
(『メタデータ情報共有のためのガイドライン 第6章 メタデータ・スキーマ定義言語』)
published (circa 2011) on the website of the **Ministry of Internal Affairs and Communications of Japan (総務省)**.
The original Japanese text is preserved verbatim in the [簡易DSP 仕様書（日本語原文）](../spec-original-ja) page.
{{< /callout >}}

SimpleDSP is a tab-separated text format for defining metadata Description Templates (Description Set Profiles). Originally specified as part of the [MetaBridge](https://metabridge.jp/) registry — built under the *Metadata Information Infrastructure Construction Project* (メタデータ情報基盤構築事業, 2010–2011) and subsequently carried on by the [Metadata Information Infrastructure Initiative (MI3)](https://web.archive.org/web/20200209202846/http://mi3.or.jp/) (メタデータ基盤協議会) — it provides a human-readable way to express which properties apply to a record, their cardinality, value types, and constraints.

SimpleDSP is the tabular serialization of the [OWL-DSP](https://www.kanzaki.com/ns/dsp#) (Description Set Profile Definition Language) ontology, where Description Templates (`dsp:DescriptionTemplate`) define record structures and Statement Templates (`dsp:StatementTemplate`) define individual property constraints. See the [OWL-DSP Specification](/specs/owl-dsp/spec/) for the full OWL-DSP ontology reference.

SimpleDSP files can be authored in a plain text editor (as `.tsv`) or in spreadsheet software (as `.csv` or `.xlsx`).

---

## 1. File Structure (§6.2.1)

- SimpleDSP is described as a text file with **tab-separated values**. The character encoding must be **UTF-8**.
- The file consists of one or more **Description Template blocks** and an optional **Namespace Declaration block**.
- Blocks begin with a **block ID** (Description Template ID) enclosed in square brackets `[]`. The namespace declaration block uses the reserved ID `@NS`.
- Blank lines may be inserted for readability; they are ignored during processing.
- Lines beginning with `#` are **comment lines** and are completely ignored during processing. A `#` in the middle of a line does **not** make the remainder a comment. This means header/label rows (which conventionally begin with `#`) are comments — column meaning is determined by position, not by header text.

**The first Description Template ID must be `MAIN`.**

If using qualified names (namespace prefixes) other than the standard ones, place a namespace declaration block `[@NS]` before the `[MAIN]` block.

```
[@NS]
prefix1	URI1
prefix2	URI2

[MAIN]
#Name	Property	Min	Max	ValueType	Constraint	Comment
...

[SubRecord]
#Name	Property	Min	Max	ValueType	Constraint	Comment
...
```

> **Shorthand (§6.2.6):** When no namespace declaration block (`[@NS]`) is present, the `[MAIN]` header line may be omitted. In practice this means a file with a single Description Template and no custom prefixes can consist of just the Statement Template rows.

---

## 2. Namespace Declaration Block (§6.2.3)

When using namespaces beyond the standard prefixes, place a namespace declaration block before the Description Template blocks. Write the block ID `[@NS]`, then list prefix–URI pairs separated by tabs on subsequent lines. Each row has two positional fields: prefix (position 1) and namespace URI (position 2).

```
[@NS]
#Prefix	Namespace URI
sioc	http://rdfs.org/sioc/ns#
rda	http://RDVocab.info/ElementsGr2/
```

The first line beginning with `#` is a comment (optional label row). The manga designer Excel omits this label row entirely — both forms are valid.

### Prefix Rules

- Prefixes may only use **alphanumeric characters**. The first character must not be a digit.
- Standard namespace URIs (see §7) may be re-bound to different prefixes.

### Special Prefix: `@base`

To define the Description Template's own namespace, use the prefix `@base`:

```
[@NS]
dmm	http://anise.slis.tsukuba.ac.jp/dmm/
@base	http://mdlab.slis.tsukuba.ac.jp/schema/dmm
```

> **Note:** The `@base` URI specifies the **Description Template's namespace**, not the default namespace of metadata instances generated using these templates.

---

## 3. Description Template Blocks (§6.2.2)

Write the Description Template ID inside brackets `[]`, then list the Statement Templates on subsequent lines, one per line.

### Block ID Rules

- The first block must use `MAIN` as its ID.
- Block IDs may use alphanumeric characters or CJK characters (Kanji/Kana). The first character **must not** be a digit.
- **Spaces and symbols** are not permitted in the ID.
- Additional blocks are referenced from other blocks using the `structured` value type with a `#` prefix (e.g. `#creator`).

If the first line of Statement Templates serves as column headers, it **must** begin with `#` to mark it as a comment — otherwise it would be parsed as a Statement Template row. The header labels are for human readers only; parsers identify columns by position.

---

## 4. Statement Templates — Column Definitions (§6.2.4)

Each row within a Description Template block defines one Statement Template. **Column meaning is determined by position (1st through 7th field), not by name:**

| Position | Conventional Label | OWL-DSP Mapping | Description |
|----------|-------------------|-----------------|-------------|
| 1 | Name | `rdfs:label` | Identifier for the statement |
| 2 | Property | `owl:onProperty` | Qualified property name |
| 3 | Min | `owl:minQualifiedCardinality` | Minimum occurrences |
| 4 | Max | `owl:maxQualifiedCardinality` | Maximum occurrences |
| 5 | Value Type | *(varies)* | Value type classification |
| 6 | Constraint | *(varies)* | Additional value constraints |
| 7 | Comment | `rdfs:comment` | Free-text documentation |

> **Core principle: columns are positional.** If a header row is present (e.g. `#Name	Property	Min	Max	ValueType	Constraint	Comment`), it begins with `#` and is therefore a **comment line — completely ignored by parsers.** The header exists purely as a human-readability convention. You may write any labels in any language; the parser never reads them. What determines each field's meaning is solely its position in the tab-separated row.
>
> This is why real-world files use varying header labels — `#項目規則名 プロパティ 最小 ...`, `#項目規則名 プロパティ 最少 ...`, `#Name Property Min ...` — all are equivalent comments.

Rows where position 5 (value type) is `ID` are special — they define the identity and type of the record itself (see §5).

### 4.1 Name (項目規則名)

The identifier given to the Statement Template, equivalent to field names like "Title" or "Author" in data.

- May use alphanumeric characters or CJK characters.
- The first character **must not** be a digit.
- Spaces and symbols should not be used.
- If spaces are used, they are converted to underscores (`_`) in the URI representation. Symbols are URL-encoded as `%HH` (the middle dot `・` is converted to `_`).

### 4.2 Property (プロパティの修飾名)

The RDF property used when converting the described item to RDF. Written as a qualified name:

- Format: `prefix:localName` (e.g. `schema:name`, `dcterms:title`)
- Prefixes must be either standard prefixes (§7) or declared in the `[@NS]` block.
- A property written **without a prefix** is treated as a custom vocabulary property.
- For `ID` rows, this column holds the **RDF class** of the record (e.g. `dmm:Manga`, `foaf:Document`), not a property.

### 4.3 Minimum Occurrences (最小出現回数)

| Value | Meaning |
|-------|---------|
| `0` | Optional |
| `1` | Required (at least once) |
| *n* | Required at least *n* times |
| keyword | Descriptive occurrence (see below) |

For occurrences like "mandatory if present" (あれば必須) or "recommended" (推奨), a **keyword** may be used in the minimum column (with the maximum set to `-`). These are computationally treated as optional (`0`).

### 4.4 Maximum Occurrences (最大出現回数)

| Value | Meaning |
|-------|---------|
| `1` | At most once |
| *n* | At most *n* times |
| `-` | Unbounded (no upper limit) |

### Common Cardinality Patterns

| Min | Max | Meaning |
|-----|-----|---------|
| `0` | `1` | Optional, at most once |
| `1` | `1` | Required, exactly once |
| `1` | `-` | Required, repeatable |
| `0` | `-` | No constraint (optional, any number) |
| `推奨` | `-` | Recommended (treated as optional for validation) |

### 4.5 Value Type (値タイプ)

Position 5 contains the value type. **This is a closed vocabulary, not free text.** The parser must recognize exactly these keywords:

| Japanese Keyword | English Keyword | OWL-DSP Mapping | Description |
|-----------------|----------------|-----------------|-------------|
| `ID` | `ID` | `dsp:valueURIOccurrence`, `dsp:resourceClass` | Defines the identity and type of the record itself (see §5) |
| `制約なし` | *(empty cell)* | — | No type constraint — any value is accepted |
| `文字列` | `literal` | `owl:onDataRange` | A literal (text) value, optionally constrained by datatype or picklist |
| `構造化` | `structured` | `owl:onClass` | A nested/structured value referencing another Description Template or class |
| `参照値` | `IRI` | `owl:onClass` / `dsp:inScheme` | A URI reference to an external resource |

**Original specification keywords:** The original specification (Table 15) defines only the Japanese keywords. The English equivalents (`literal`, `structured`, `IRI`) are conventions introduced by YAMA for English-language export and are not part of the original specification. The original MetaBridge platform used only the Japanese keywords.

**Parsing rules:**
- The value type is **case-insensitive** (`literal`, `Literal`, `LITERAL` are equivalent).
- YAMA accepts both Japanese and English keywords for interoperability.
- `制約なし` (the literal Japanese text) and an empty cell are equivalent; both mean "unconstrained."
- Some Excel files write `"ID"` (with quotes) — parsers should strip quotes.
- No other values are valid in position 5. An unrecognized value type is an error.

### 4.6 Value Constraint (値制約)

Position 6 contains additional constraints. The syntax depends on the value type in position 5. Below is the **complete** set of constraint patterns from the original specification (Tables 16–18).

#### For `literal` / `文字列`

Four constraint forms:

| Pattern | Constraint | OWL-DSP | Example |
|---------|------------|---------|---------|
| *(empty)* | Any string (plain literal) | `owl:onDataRange rdfs:Literal` | `title	dc:title	0	1	文字列` |
| `datatype` | Datatype restriction | `owl:onDataRange datatype` | `発行日	dcterms:issued	1	1	文字列	xsd:date` |
| `dt1 dt2` | Multiple datatypes (space-separated) | union of `owl:onDataRange` | `value	ex:amount	0	1	literal	xsd:decimal xsd:integer` |
| `"v1" "v2" "v3"` | Picklist — value must be one of the quoted strings | enumeration | `color	dc:format	1	1	literal	"red" "green" "blue"` |

Picklist examples from spec: `"バナナ" "りんご" "みかん"` — value must be one of "banana", "apple", or "mandarin".

#### For `structured` / `構造化`

Four constraint forms:

| Pattern | Constraint | OWL-DSP | Example |
|---------|------------|---------|---------|
| `#blockId` | Same-file block reference | `owl:onClass <#blockId>` | `Setting	dmm:hasSetting	0	-	構造化	#Setting` |
| `className` | Instance of this class (no property constraints) | `owl:onClass className` | `著者	dcterms:creator	0	1	構造化	foaf:Agent` |
| `class1 class2` | Instance of one of the listed classes | `owl:onClass [owl:unionOf(...)]` | `Creator	dc:creator	0	1	structured	foaf:Person rda:Family` |

> To apply both a class constraint **and** property constraints to a structured value, use the `#blockId` form and specify the class in the referenced block's ID statement (Property column).
>
> Defining an anonymous (blank node) class constraint without an ID row is not supported in SimpleDSP (it can be done directly in OWL-DSP).
>
> Self-references are valid: `#Setting` can reference the `[Setting]` block it appears in, creating a recursive structure.

#### For `IRI` / `参照値`

Four constraint forms:

| Pattern | Constraint | OWL-DSP | Example |
|---------|------------|---------|---------|
| *(empty)* | Any URI | `owl:onClass [owl:oneOf [rdfs:Resource]]` | `image	dmm:image	0	-	参照値` |
| `prefix:` | URIs from this vocabulary (trailing `:`) | `owl:onClass [dsp:inScheme prefix:]` | `主題	dcterms:subject	0	-	参照値	ndlsh:` |
| `pfx1: pfx2:` | URIs from any listed vocabulary | `owl:onClass [owl:unionOf([dsp:inScheme ...])]` | `主題	dcterms:subject	0	-	参照値	ndlsh: bsh:` |
| `pfx:name` | Specific URI(s) (qualified names) | `owl:onClass [owl:oneOf ...]` | `payment	ex:method	0	1	IRI	card:VISA card:AMEX` |

> **Note on OWL-DSP mappings:** The OWL-DSP column values for `IRI` (参照値) constraints are derived from the MetaBridge registry implementation output, not explicitly defined in the original specification text. The original spec defines the SimpleDSP syntax; the OWL-DSP mappings show how MetaBridge converts them.
Full URIs can also be specified in angle brackets: `<http://example.org/thing1> <http://example.org/thing2>`.

> **Note:** A constraint like `ndlsh:` means data values should be URIs belonging to the NDLSH vocabulary namespace (e.g. `ndlsh:図書館`), not the prefix itself. Also note that while individual URIs are generally expected to belong to the declared namespace, this is not guaranteed — the namespace is a scheme identifier, not a strict filter.

#### For *(empty)* / `制約なし`

The constraint column is typically left empty as well. No constraint is applied.

### 4.7 Comment (コメント)

Free-text documentation for the item. Used as help text or notes.

---

## 5. ID Statement (§6.2.4 — ID項目規則)

The ID Statement Template is a special row that defines the identity and handling of the record itself. It maps to `dsp:valueURIOccurrence` and `dsp:resourceClass` in OWL-DSP.

| Column | Value | Description |
|--------|-------|-------------|
| Name | Field name | Name of the identifier field (e.g. `MangaID`, `書誌ID`) |
| Property | Class name | RDF class of the record (e.g. `dmm:Manga`, `foaf:Document`) — **not** a property |
| Min | `1` | Always `1` |
| Max | `1` | Always `1` |
| ValueType | `ID` | Always `ID` (may appear as `"ID"` in some Excel files) |
| Constraint | Base URI *(optional)* | Namespace for constructing record URIs |
| Comment | Description | Description of the record type |

**With base URI constraint** (NDL example):
```
書誌ID	foaf:Document	1	1	ID	ndlbooks:	文書のID
```
A data value `b01234` produces the full record URI `ndlbooks:b01234`.

**Without base URI constraint** (manga designer example):
```
MangaID	dmm:Manga	1	1	ID
```
The record URI is determined externally (by the registry or application). The original spec notes that specifying a base URI fixes how the description rule is used.

**Rules:**
- Each Description Template block may have **at most one** ID statement.
- The `[MAIN]` block should, as a rule, include an ID statement.
- The cardinality for ID rows is always fixed at `1`/`1`.

---

## 6. Worked Examples

Complete SimpleDSP profiles with rendered tables and downloadable files are available in the [Examples](../examples/) section:

{{< cards >}}
  {{< card link="../examples/ndl-bibliographic" title="NDL Bibliographic Record" subtitle="Canonical example from the original specification §6.2.5" icon="book-open" >}}
  {{< card link="../examples/manga-design" title="Manga Design Metadata" subtitle="Real-world multi-block SimpleDSP from the Digital Manga Model project" icon="document-text" >}}
  {{< card link="../examples/tbbt-characters" title="TBBT Characters" subtitle="Small profile shown as TSV, CSV, and XLSX — demonstrates format equivalence" icon="table" >}}
{{< /cards >}}


---

## 7. Standard Namespace Prefixes (§6.2.6)

A SimpleDSP processor maintains the following standard prefix table. Prefixes in this table are consulted as a fallback to resolve any CURIE whose prefix is not declared in the document's `[@NS]` block.

| Prefix | Namespace URI | Source |
|--------|---------------|--------|
| `dc` | `http://purl.org/dc/elements/1.1/` | Original spec |
| `dcterms` | `http://purl.org/dc/terms/` | Original spec |
| `foaf` | `http://xmlns.com/foaf/0.1/` | Original spec |
| `skos` | `http://www.w3.org/2004/02/skos/core#` | Original spec |
| `xl` | `http://www.w3.org/2008/05/skos-xl#` | Original spec |
| `rdf` | `http://www.w3.org/1999/02/22-rdf-syntax-ns#` | Original spec |
| `rdfs` | `http://www.w3.org/2000/01/rdf-schema#` | Original spec |
| `owl` | `http://www.w3.org/2002/07/owl#` | Original spec |
| `xsd` | `http://www.w3.org/2001/XMLSchema#` | Original spec |
| `schema` | `https://schema.org/` | YAMA extension |

> **Note on `schema:`** The original specification (Table 19) defines 9 standard prefixes. The `schema` prefix for Schema.org was not included because the vocabulary did not exist when the specification was written. YAMA adds `schema` as a 10th standard prefix to support the majority of modern application profiles that rely on Schema.org vocabulary.

### Resolution rule (YAMA extension)

For any CURIE `prefix:localPart` in a SimpleDSP document, a YAMA-compliant processor resolves the prefix as follows:

1. If `prefix` is declared in the document's `[@NS]` block, use the URI bound there.
2. Otherwise, if `prefix` appears in the standard prefix table above, use the URI from the table.
3. Otherwise, the prefix is undeclared and the document is not well-formed.

The user's `[@NS]` declarations take precedence unconditionally. A prefix declared in `[@NS]` may bind to any URI; the processor does not compare it against the standard table. This means:

- Authors may redefine a standard prefix to point to a different URI if that suits their profile's local conventions.
- Authors may use an unconventional label (for example, `DCTERMS` instead of `dcterms`, or `dc` for the dcterms namespace) by declaring it in `[@NS]`.
- Prefix labels in RDF/SHACL/ShEx outputs mirror those used in the profile — exported `@prefix` declarations are derived from the resolved bindings, not renamed to canonical forms.

This resolution rule is a YAMA extension that formalises a behaviour left implicit in the original 2011 specification. The original text permitted redeclaration of standard prefixes in `[@NS]` but did not define precedence; YAMA fixes this by giving the user's declaration the decisive say.

### Omitting `[@NS]`

If every CURIE in the document resolves through the standard prefix table and no overriding declarations are needed, the `[@NS]` block may be omitted entirely.

---

## 8. File Formats

SimpleDSP can be stored in three file formats. The original specification defines only the tab-separated text format; CSV and Excel support are YAMA extensions for interoperability.

### Tab-Separated Text (`.tsv`)

The native format. Fields are separated by tab characters. Block markers (`[@NS]`, `[MAIN]`, etc.) appear as the first field in their row.

### Comma-Separated Values (`.csv`)

Standard CSV format (RFC 4180). Fields containing commas, quotes, or newlines must be enclosed in double quotes. Block markers appear as single-cell rows.

### Microsoft Excel (`.xlsx`)

A single worksheet containing the same structure as the TSV format. Block markers appear in the first cell of their row. Each field occupies one cell.

### Equivalence across formats

The three file formats carry identical information. See the [worked example](../examples/tbbt-characters/) which presents the same SimpleDSP profile as raw TSV, as a rendered table, and as downloadable files in all three formats.

The `.xlsx` files published with YAMA are styled with subtle coloring for readability — block headers appear in a dark slate band, the ID row is highlighted in warm amber, and data rows alternate between white and a very light grey. The styling is purely visual; it carries no semantic meaning and does not affect how the file is parsed.

---

## 9. OWL-DSP Mapping Reference

SimpleDSP is the tabular serialization of the [OWL-DSP](/specs/owl-dsp/spec/) ontology. The following table summarizes the mapping:

| SimpleDSP Element | OWL-DSP | URI |
|---|---|---|
| Block (e.g. `[MAIN]`) | Description Template | `dsp:DescriptionTemplate` |
| Row (Statement) | Statement Template | `dsp:StatementTemplate` |
| Name column | Label | `rdfs:label` |
| Property column | on Property | `owl:onProperty` |
| Min column | min Qualified Cardinality | `owl:minQualifiedCardinality` |
| Max column | max Qualified Cardinality | `owl:maxQualifiedCardinality` |
| Cardinality keyword | Cardinality Note | `dsp:cardinalityNote` |
| ValueType = `literal` | on Data Range | `owl:onDataRange` (→ `rdfs:Literal` if no constraint) |
| ValueType = `structured` (`#ref`) | on Class | `owl:onClass <#ref>` |
| ValueType = `structured` (class) | on Class | `owl:onClass className` |
| ValueType = `IRI` (unconstrained) | on Class | `owl:onClass [owl:oneOf [rdfs:Resource]]` |
| ValueType = `IRI` (vocab) | in Scheme | `owl:onClass [dsp:inScheme vocab:]` |
| ValueType = `IRI` (specific URI) | on Class | `owl:onClass [owl:oneOf ...]` |
| ValueType = `ID` (Property col) | Resource Class | `dsp:resourceClass` |
| ValueType = `ID` (Constraint col) | Value URI Occurrence | `dsp:valueURIOccurrence "mandatory"` |
| Comment column | comment | `rdfs:comment` |

For a complete OWL-DSP ontology reference with worked examples, see the [OWL-DSP Specification](/specs/owl-dsp/spec/).

---

## 10. CLI Usage

The `yama` CLI supports SimpleDSP as both input and output format across multiple commands.

### Export YAMA to SimpleDSP

```sh
yama simpledsp -i input.yaml -o output.tsv
yama simpledsp -i input.yaml -o output.csv
yama simpledsp -i input.yaml -o output.xlsx
```

Use `-l jp` (or `--lang jp`) to output Japanese column headers and value type names (the original specification language):

```sh
yama simpledsp -i input.yaml -o output.tsv -l jp
```

### Import SimpleDSP to YAMA

Both English and Japanese headers/value types are accepted automatically:

```sh
yama from-simpledsp -i input.tsv -o output.yaml
yama from-simpledsp -i input.csv -o output.yaml
yama from-simpledsp -i input.xlsx -o output.yaml
```

### Validate a SimpleDSP File

```sh
yama validate -i profile.tsv
yama validate -i profile.xlsx
yama validate -i profile.tsv --format json    # JSON output for CI
```

### Generate Documentation from SimpleDSP

```sh
yama report -i profile.tsv -o profile.html    # Standalone HTML report
yama report -i profile.tsv -o profile.md      # Markdown
yama package -i profile.tsv -o dist/          # Full artifact package
```

---

## References

- [*Guidelines for Metadata Information Sharing* (メタデータ情報共有のためのガイドライン), Chapter 6, Metadata Information Infrastructure Initiative (MI3 — メタデータ基盤協議会)](https://www.soumu.go.jp/main_content/000132512.pdf)
- MetaBridge Help: [SimpleDSP File Format](https://web.archive.org/web/20220817110605/https://metabridge.jp/help/help_dsp.html#dsp_edit) (Japanese, archived)
- Kanzaki, M. *OWL-DSP — Description Set Profile Definition Language*: <https://www.kanzaki.com/ns/dsp#>
- OWL-DSP Ontology namespace: `http://purl.org/metainfo/terms/dsp#`
- NDL Current Awareness-R (2011-09-21): *メタデータ情報基盤構築事業で構築されたメタデータレジストリ "Meta Bridge"* — <https://current.ndl.go.jp/car/19135>

## Related Publications

- Nagamori, M., Kanzaki, M., Torigoshi, N., & Sugimoto, S. (2011). **Meta-Bridge: A Development of Metadata Information Infrastructure in Japan**. In *Proceedings of the International Conference on Dublin Core and Metadata Applications* (Vol. 2011). Dublin Core Metadata Initiative. <https://doi.org/10.23106/dcmi.952135745>
