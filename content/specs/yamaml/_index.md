---
title: YAMAML
weight: 1
sidebar:
  open: true
---

**YAMAML** (Yet Another Metadata Application Profile and RDF Mapping Language) is the core YAML-native language of the YAMA family (circa 2019). It is a human-friendly markup language for creating, managing, and publishing metadata application profiles.

A YAMAML document describes the structure and constraints of metadata records using YAML 1.2 syntax, and can be transformed into RDF, SHACL, ShEx, OWL-DSP, SimpleDSP, DCTAP, and Frictionless Data Packages.

{{< cards >}}
  {{< card link="spec" title="Full Specification" subtitle="The complete YAMAML specification, version 0.2.0" icon="document-text" >}}
  {{< card link="pkl" title="Authoring in PKL" subtitle="Type-safe authoring layer that renders to YAMAML" icon="code" >}}
  {{< card link="examples" title="Examples" subtitle="Worked YAMAML profiles you can study and download" icon="book-open" >}}
{{< /cards >}}

## At a Glance

- **Human-readable** — author in a plain text editor
- **YAML-native** — parsable by any YAML 1.2 parser
- **Multi-target** — one source, many outputs
- **Extensible** — custom keys are permitted
- **Practical** — optional data mapping for RDF generation

## Authoring Layers

YAMAML is the canonical source of truth for the language semantics, but you can author profiles in multiple surface syntaxes that all render to the same underlying model:

- **YAMAML (YAML)** — the default, canonical format. Simple, portable, works everywhere.
- **PKL** — Apple's typed configuration language. Ideal for large profiles that benefit from strong typing, editor autocomplete, and modular composition. [Learn more →](pkl)
