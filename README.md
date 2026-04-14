# YAMA Documentation

Documentation site for the YAMA family of metadata application profile languages:
YAMAML, SimpleDSP, and OWL-DSP.

**URL:** https://docs.yamaml.org

Built with [Hugo](https://gohugo.io/) and the [Hextra](https://imfing.github.io/hextra/) theme (v0.12.1, via Hugo Modules).

## Structure

```
content/
├── _index.md              # Landing page
├── about.md               # About page
└── specs/
    ├── _index.md          # Specifications hub
    ├── yamaml/
    │   ├── _index.md      # YAMAML section intro
    │   ├── spec.md        # Full YAMAML specification
    │   ├── pkl.md         # PKL authoring guide
    │   └── examples/      # Worked YAMAML profiles
    ├── simpledsp/
    │   ├── _index.md      # SimpleDSP section intro
    │   ├── spec.md        # English specification
    │   ├── spec-original-ja.md   # Japanese original (verbatim)
    │   └── examples/      # Worked SimpleDSP profiles (NDL, manga, TBBT)
    └── owl-dsp/
        ├── _index.md      # OWL-DSP section intro
        ├── spec.md        # English specification
        └── spec-original-ja.md   # Japanese original (verbatim)

i18n/
└── en.yaml                # Footer copyright override

static/
└── examples/simpledsp/    # Downloadable SimpleDSP examples (TSV, CSV, XLSX)
```

## Local Development

Prerequisites: [Hugo](https://gohugo.io/getting-started/installing/) (extended edition), [Go](https://golang.org/doc/install)

```sh
# Download the Hextra theme module
hugo mod tidy

# Start the dev server
hugo server --port 8081 --disableFastRender
```

The site will be available at http://localhost:8081.

## Specifications

| Spec | English | Japanese original |
|------|---------|-------------------|
| YAMAML | `specs/yamaml/spec` | — |
| SimpleDSP | `specs/simpledsp/spec` | `specs/simpledsp/spec-original-ja` |
| OWL-DSP | `specs/owl-dsp/spec` | `specs/owl-dsp/spec-original-ja` |

## License

The YAMA specifications are licensed under
[Creative Commons Attribution-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-sa/4.0/).
