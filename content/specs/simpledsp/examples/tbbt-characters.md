---
title: TBBT Characters
weight: 3
toc: true
---

A small SimpleDSP profile describing characters from *The Big Bang Theory* — a `[MAIN]` description for a person with an embedded structured `[address]` sub-description. The same content is shown first as raw TSV, then rendered as a table for easier reading, and finally as downloadable files ready to open in any spreadsheet application. Use this example to confirm that the three supported file formats (TSV, CSV, XLSX) carry identical information.

## Raw TSV

```tsv {filename="tbbt-characters.tsv"}
[@NS]
schema	http://schema.org/
@base	http://purl.org/yama/examples/2022/tbbt/0.1/

[MAIN]
#Name	Property	Min	Max	ValueType	Constraint	Comment
ID	foaf:Person	1	1	ID		Unique identifier for the character
Name	foaf:name	1	1	literal	xsd:string	Full name of the character
Family Name	foaf:familyName	1	1	literal	xsd:string	Family name
First Name	foaf:firstName	1	1	literal	xsd:string	Given name
Job Title	schema:jobTitle	0	1	literal	xsd:string	Job title of the character
Parents	schema:parent	0	-	IRI		Parents of the character
Children	schema:children	0	-	IRI		Children of the character
Knows	foaf:knows	0	-	IRI		Other characters this character knows
Wikidata	rdfs:seeAlso	0	-	IRI		Wikidata entity for this character
Home Address	schema:address	0	1	structured	#address	Home address of the character
Portrayed by	schema:byArtist	0	1	IRI		Actor who portrays this character

[address]
#Name	Property	Min	Max	ValueType	Constraint	Comment
Street	schema:streetAddress	0	1	literal	xsd:string	Building and street address
Locality	schema:addressLocality	0	1	literal	xsd:string	City or town
Region	schema:addressRegion	0	1	literal	xsd:string	State, province, or region
Country	schema:addressCountry	0	1	literal	xsd:string	Country
Postal Code	schema:postalCode	0	1	literal	xsd:string	Postal or ZIP code
```

## Rendered as a table

The `[@NS]` namespace block declares one custom prefix and the base URI:

| Prefix | Namespace URI |
|---|---|
| `schema` | `http://schema.org/` |
| `@base` | `http://purl.org/yama/examples/2022/tbbt/0.1/` |

The `[MAIN]` description block defines the character schema:

| Name | Property | Min | Max | ValueType | Constraint | Comment |
|---|---|:---:|:---:|---|---|---|
| **ID** | `foaf:Person` | 1 | 1 | **ID** | | Unique identifier for the character |
| Name | `foaf:name` | 1 | 1 | literal | `xsd:string` | Full name of the character |
| Family Name | `foaf:familyName` | 1 | 1 | literal | `xsd:string` | Family name |
| First Name | `foaf:firstName` | 1 | 1 | literal | `xsd:string` | Given name |
| Job Title | `schema:jobTitle` | 0 | 1 | literal | `xsd:string` | Job title of the character |
| Parents | `schema:parent` | 0 | - | IRI | | Parents of the character |
| Children | `schema:children` | 0 | - | IRI | | Children of the character |
| Knows | `foaf:knows` | 0 | - | IRI | | Other characters this character knows |
| Wikidata | `rdfs:seeAlso` | 0 | - | IRI | | Wikidata entity for this character |
| Home Address | `schema:address` | 0 | 1 | structured | `#address` | Home address of the character |
| Portrayed by | `schema:byArtist` | 0 | 1 | IRI | | Actor who portrays this character |

The `[address]` sub-description is used as a structured value from `Home Address` above:

| Name | Property | Min | Max | ValueType | Constraint | Comment |
|---|---|:---:|:---:|---|---|---|
| Street | `schema:streetAddress` | 0 | 1 | literal | `xsd:string` | Building and street address |
| Locality | `schema:addressLocality` | 0 | 1 | literal | `xsd:string` | City or town |
| Region | `schema:addressRegion` | 0 | 1 | literal | `xsd:string` | State, province, or region |
| Country | `schema:addressCountry` | 0 | 1 | literal | `xsd:string` | Country |
| Postal Code | `schema:postalCode` | 0 | 1 | literal | `xsd:string` | Postal or ZIP code |

Note that the `[address]` block has no ID row — it is only ever used as an inline structured value, so its instances do not need their own record identifiers.

## Downloads

The same profile is available as ready-to-use files in all three formats:

{{< cards >}}
  {{< card link="/examples/simpledsp/tbbt-characters.tsv" title="tbbt-characters.tsv" subtitle="Tab-separated (native format)" icon="document-text" >}}
  {{< card link="/examples/simpledsp/tbbt-characters.csv" title="tbbt-characters.csv" subtitle="Comma-separated (RFC 4180)" icon="document-text" >}}
  {{< card link="/examples/simpledsp/tbbt-characters.xlsx" title="tbbt-characters.xlsx" subtitle="Microsoft Excel (styled)" icon="table" >}}
{{< /cards >}}

The `.xlsx` file is styled with subtle coloring for readability — block headers appear in a dark slate band, the ID row is highlighted in warm amber, and data rows alternate between white and a very light grey. The styling is purely visual; it carries no semantic meaning and does not affect how the file is parsed.
