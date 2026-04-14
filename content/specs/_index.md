---
title: Specifications
next: specs/yamaml
sidebar:
  open: true
---

YAMA-related specifications for describing metadata application profiles:

{{< cards >}}
  {{< card link="yamaml" title="YAMAML" subtitle="The core YAML-native source language. Write once, generate every supported format." icon="book-open" >}}
  {{< card link="simpledsp" title="SimpleDSP" subtitle="A compact tab-separated format for describing record structures. Originally specified in Japanese as 簡易DSP." icon="table" >}}
  {{< card link="owl-dsp" title="OWL-DSP" subtitle="An OWL ontology for describing Dublin Core Description Set Profiles. The formal semantic model behind the family." icon="document-text" >}}
{{< /cards >}}

## About the Specifications

- **YAMAML** is the primary authoring language. It uses YAML 1.2 syntax and supports every feature of the other formats plus data mapping for RDF generation.
- **SimpleDSP** (簡易DSP) is a long-established tab-separated format from the [Metadata Information Sharing Guidelines](https://www.soumu.go.jp/main_content/000132512.pdf) (circa 2011). YAMA preserves full fidelity with the original specification, available in both English and Japanese.
- **OWL-DSP** is the underlying ontology used for rich semantic descriptions. Also available in the original Japanese.

{{< callout type="info" >}}
  The Japanese originals are preserved verbatim in a separate language section for readers who prefer to work with the authoritative source text.
{{< /callout >}}
