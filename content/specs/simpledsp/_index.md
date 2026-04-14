---
title: SimpleDSP
weight: 2
sidebar:
  open: true
---

**SimpleDSP** (簡易DSP) is a compact tab-separated format for describing record structures. It was originally specified in Japanese as part of the [*Metadata Information Sharing Guidelines, Chapter 6*](https://www.soumu.go.jp/main_content/000132512.pdf), produced circa 2011 under the *Metadata Information Infrastructure Construction Project* (メタデータ情報基盤構築事業), whose work was subsequently carried on by the [Metadata Information Infrastructure Initiative (MI3)](https://web.archive.org/web/20200209202846/http://mi3.or.jp/) (メタデータ基盤協議会, Japan).

YAMA's SimpleDSP implementation preserves full fidelity with the original specification, with an English translation available alongside the Japanese original.

{{< cards >}}
  {{< card link="spec" title="SimpleDSP Specification (English)" subtitle="Full English specification based on the Japanese original" icon="document-text" >}}
  {{< card link="spec-original-ja" title="簡易DSP 仕様書（日本語原文）" subtitle="原典である日本語版仕様書" icon="document-text" >}}
  {{< card link="examples" title="Examples" subtitle="Worked SimpleDSP profiles with TSV, CSV, and XLSX downloads" icon="book-open" >}}
{{< /cards >}}

## At a Glance

- Tab-separated text format with UTF-8 encoding
- Description blocks delimited by `[BlockID]` headers
- Namespace declarations in a `[@NS]` block
- Seven columns per statement: Name, Property, Min, Max, ValueType, Constraint, Comment
- Value types: `ID`, `literal`, `structured`, `IRI`, empty (unconstrained)
