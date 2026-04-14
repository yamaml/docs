---
title: OWL-DSP
weight: 3
sidebar:
  open: true
---

**OWL-DSP** is an OWL ontology for describing **Description Set Profiles** (DSPs) — the conceptual model behind the YAMA family. It was originally specified as part of the [Metadata Information Sharing Guidelines](https://www.soumu.go.jp/main_content/000132512.pdf), produced circa 2011 under the *Metadata Information Infrastructure Construction Project* (メタデータ情報基盤構築事業), whose work was subsequently carried on by the [Metadata Information Infrastructure Initiative (MI3)](https://web.archive.org/web/20200209202846/http://mi3.or.jp/) (メタデータ基盤協議会, Japan). It provides the formal semantic backbone used to express metadata application profiles with rich semantics.

{{< cards >}}
  {{< card link="spec" title="OWL-DSP Specification (English)" subtitle="Full English specification based on the Japanese original" icon="document-text" >}}
  {{< card link="spec-original-ja" title="OWL-DSP 仕様書（日本語原文）" subtitle="原典である日本語版仕様書" icon="document-text" >}}
{{< /cards >}}

## At a Glance

- An OWL 2 ontology with namespace `https://www.kanzaki.com/ns/dsp#`
- Describes `DescriptionTemplate` (shape) and `StatementTemplate` (property constraint)
- Supports cardinality, value types, datatype, class, shape reference, vocabulary scheme, picklists, and cardinality notes
- Used by YAMAML as the rich semantic target format
