---
title: Manga Design Metadata
weight: 2
toc: true
---

A real-world multi-block SimpleDSP from the Digital Manga Model (DMM) project at the University of Tsukuba, defining a schema for manga design analysis.

This example uses Japanese value types and demonstrates self-referencing blocks, multiple levels of nested structure, and ID rows without base URI constraints.

## Raw TSV

```
[@NS]
foaf	http://xmlns.com/foaf/0.1/
dc	http://purl.org/dc/elements/1.1/
dct	http://purl.org/dc/terms/
dmm	http://anise.slis.tsukuba.ac.jp/dmm/
rdfs	http://www.w3.org/2000/01/rdf-schema#

[MAIN]
#項目規則名	プロパティ	最少	最大	値タイプ	値制約	コメント
MangaID	dmm:Manga	1	1	ID
Setting	dmm:hasSetting	0	-	構造化	#Setting	マンガの設定

[Setting]
SettingID	dmm:Setting	1	1	ID
title	dc:title	0	1	文字列
image	dmm:image	0	-	参照値
description	dc:description	0	1	文字列
relation	dmm:relation	0	-	構造化	#Setting
CharacterSetting	dmm:hasCharacterSetting	0	1	構造化	#CharacterSetting	人物設定
StageSetting	dmm:hasStageSetting	0	1	構造化	#StageSetting	舞台設定
PlotSetting	dmm:hasPlotSetting	0	1	構造化	#PlotSetting	事柄設定

[CharacterSetting]
CharacterSettingID	dmm:CharacterSetting	1	1	ID
name	dmm:name	0	-	文字列
sex	dmm:sex	0	-	文字列
hometown	dmm:hometown	0	-	文字列
model	dmm:model	0	-	参照値
role	dmm:hasRole	0	-	文字列

[PlotSetting]
PlotSettingID	dmm:StorySetting	1	1	ID
cast	dmm:hasCarahcter	0	-	構造化	#CharacterSetting
stage	dmm:hasStage	0	-	構造化	#StageSetting

[StageSetting]
StageSettingID	dmm:StageSetting	1	1	ID
location	dmm:name	0	-	文字列
coord	dmm:coord	0	-	文字列

[Role]
RoleID	dmm:Role	1	1	ID
RoleName	dmm:name	0	-	文字列
description	dc:description	0	-	文字列
```

## Rendered as tables

**Namespaces** — one custom prefix (`dmm`), four standard-prefix redeclarations, and no `@base`:

| Prefix | Namespace URI |
|---|---|
| `foaf` | `http://xmlns.com/foaf/0.1/` |
| `dc` | `http://purl.org/dc/elements/1.1/` |
| `dct` | `http://purl.org/dc/terms/` |
| `dmm` | `http://anise.slis.tsukuba.ac.jp/dmm/` |
| `rdfs` | `http://www.w3.org/2000/01/rdf-schema#` |

**[MAIN]** — the top-level description template:

| 項目規則名 | プロパティ | 最少 | 最大 | 値タイプ | 値制約 | コメント |
|---|---|:---:|:---:|---|---|---|
| **MangaID** | `dmm:Manga` | 1 | 1 | **ID** | | |
| Setting | `dmm:hasSetting` | 0 | - | 構造化 | `#Setting` | マンガの設定 |

**[Setting]** — a self-referencing structured value with four nested sub-descriptions:

| 項目規則名 | プロパティ | 最少 | 最大 | 値タイプ | 値制約 | コメント |
|---|---|:---:|:---:|---|---|---|
| **SettingID** | `dmm:Setting` | 1 | 1 | **ID** | | |
| title | `dc:title` | 0 | 1 | 文字列 | | |
| image | `dmm:image` | 0 | - | 参照値 | | |
| description | `dc:description` | 0 | 1 | 文字列 | | |
| relation | `dmm:relation` | 0 | - | 構造化 | `#Setting` | |
| CharacterSetting | `dmm:hasCharacterSetting` | 0 | 1 | 構造化 | `#CharacterSetting` | 人物設定 |
| StageSetting | `dmm:hasStageSetting` | 0 | 1 | 構造化 | `#StageSetting` | 舞台設定 |
| PlotSetting | `dmm:hasPlotSetting` | 0 | 1 | 構造化 | `#PlotSetting` | 事柄設定 |

**[CharacterSetting]** — describes a character in the manga:

| 項目規則名 | プロパティ | 最少 | 最大 | 値タイプ | 値制約 | コメント |
|---|---|:---:|:---:|---|---|---|
| **CharacterSettingID** | `dmm:CharacterSetting` | 1 | 1 | **ID** | | |
| name | `dmm:name` | 0 | - | 文字列 | | |
| sex | `dmm:sex` | 0 | - | 文字列 | | |
| hometown | `dmm:hometown` | 0 | - | 文字列 | | |
| model | `dmm:model` | 0 | - | 参照値 | | |
| role | `dmm:hasRole` | 0 | - | 文字列 | | |

**[PlotSetting]** — a story element that cross-references characters and stages:

| 項目規則名 | プロパティ | 最少 | 最大 | 値タイプ | 値制約 | コメント |
|---|---|:---:|:---:|---|---|---|
| **PlotSettingID** | `dmm:StorySetting` | 1 | 1 | **ID** | | |
| cast | `dmm:hasCarahcter` | 0 | - | 構造化 | `#CharacterSetting` | |
| stage | `dmm:hasStage` | 0 | - | 構造化 | `#StageSetting` | |

**[StageSetting]** — describes a location or setting:

| 項目規則名 | プロパティ | 最少 | 最大 | 値タイプ | 値制約 | コメント |
|---|---|:---:|:---:|---|---|---|
| **StageSettingID** | `dmm:StageSetting` | 1 | 1 | **ID** | | |
| location | `dmm:name` | 0 | - | 文字列 | | |
| coord | `dmm:coord` | 0 | - | 文字列 | | |

**[Role]** — defined but never referenced (unused block):

| 項目規則名 | プロパティ | 最少 | 最大 | 値タイプ | 値制約 | コメント |
|---|---|:---:|:---:|---|---|---|
| **RoleID** | `dmm:Role` | 1 | 1 | **ID** | | |
| RoleName | `dmm:name` | 0 | - | 文字列 | | |
| description | `dc:description` | 0 | - | 文字列 | | |

## Downloads

{{< cards >}}
  {{< card link="/examples/simpledsp/manga-design.tsv" title="manga-design.tsv" subtitle="Tab-separated (native format)" icon="document-text" >}}
  {{< card link="/examples/simpledsp/manga-design.csv" title="manga-design.csv" subtitle="Comma-separated (RFC 4180)" icon="document-text" >}}
  {{< card link="/examples/simpledsp/manga-design.xlsx" title="manga-design.xlsx" subtitle="Microsoft Excel (styled)" icon="table" >}}
{{< /cards >}}
