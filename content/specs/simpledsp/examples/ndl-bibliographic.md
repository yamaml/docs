---
title: NDL Bibliographic Record
weight: 1
toc: true
---

This is the canonical example from the original Japanese SimpleDSP specification (§6.2.5), defining a bibliographic record for the National Diet Library with structured (nested) title and vocabulary-constrained subject headings.

## Raw TSV

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

This example demonstrates:
- **ID statement:** `書誌ID` row defines records as `foaf:Document` instances with `ndlbooks:` namespace URIs
- **Structured value with block reference:** `タイトル` references the `[構造化タイトル]` block via `#構造化タイトル`
- **Structured value with class constraint:** `著者` constrains values to `foaf:Agent` instances without a separate block
- **Literal with datatype:** `発行日` constrains values to `xsd:date`
- **Reference with vocabulary schemes:** `主題` constrains values to URIs from `ndlsh:` or `bsh:` vocabularies
- **Japanese column headers and value types:** The format supports both Japanese and English terms

## Rendered as tables

**Namespaces** — four custom prefixes plus an `@base` URI for the schema itself:

| Prefix | Namespace URI |
|---|---|
| `dcndl` | `http://ndl.go.jp/dcndl/terms/` |
| `ndlsh` | `http://id.ndl.go.jp/auth/ndlsh/` |
| `bsh` | `http://id.ndl.go.jp/auth/bsh/` |
| `ndlbooks` | `http://iss.ndl.go.jp/books/` |
| `@base` | `http://ndl.go.jp/dcndl/dsp/biblio` |

**[MAIN]** — the bibliographic record template with an `ndlbooks:` ID namespace:

| 項目規則名 | プロパティ | 最小 | 最大 | 値タイプ | 値制約 | 説明 |
|---|---|:---:|:---:|---|---|---|
| **書誌ID** | `foaf:Document` | 1 | 1 | **ID** | `ndlbooks:` | 文書のID |
| タイトル | `dcterms:title` | 1 | 1 | 構造化 | `#構造化タイトル` | 文書の表題 |
| 著者 | `dcterms:creator` | 0 | 1 | 構造化 | `foaf:Agent` | 文書の作者 |
| 発行日 | `dcterms:issued` | 1 | 1 | 文字列 | `xsd:date` | 文書の発行日 |
| 主題 | `dcterms:subject` | 0 | - | 参照値 | `ndlsh: bsh:` | 文書の主題 |

**[構造化タイトル]** — a nested structured value for the title with its own reading:

| 項目規則名 | プロパティ | 最小 | 最大 | 値タイプ | 値制約 | 説明 |
|---|---|:---:|:---:|---|---|---|
| リテラル値 | `xl:literalForm` | 1 | 1 | 文字列 | | タイトル自身 |
| 読み | `dcndl:transcription` | 0 | 1 | 文字列 | | タイトルの読み |

## Downloads

{{< cards >}}
  {{< card link="/examples/simpledsp/ndl-bibliographic.tsv" title="ndl-bibliographic.tsv" subtitle="Tab-separated (native format)" icon="document-text" >}}
  {{< card link="/examples/simpledsp/ndl-bibliographic.csv" title="ndl-bibliographic.csv" subtitle="Comma-separated (RFC 4180)" icon="document-text" >}}
  {{< card link="/examples/simpledsp/ndl-bibliographic.xlsx" title="ndl-bibliographic.xlsx" subtitle="Microsoft Excel (styled)" icon="table" >}}
{{< /cards >}}
