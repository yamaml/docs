---
title: OWL-DSP 仕様書（日本語原文）
weight: 2
toc: true
description: "OWL-DSP 仕様書の日本語原文。総務省のウェブサイトで公開されている『メタデータ情報共有のためのガイドライン 第6章』セクション 6.1 および 6.3 を原文のまま忠実に記録。"
---

{{< callout type="info" >}}
この文書は、[**総務省のウェブサイトで公開されている**](https://www.soumu.go.jp/main_content/000132512.pdf)（2011 年頃公開）
『メタデータ情報共有のためのガイドライン 第6章 メタデータ・スキーマ定義言語』
（メタデータ情報基盤構築事業、メタデータ基盤協議会）セクション 6.1「記述規則定義言語」
および 6.3「OWL記述例」を原文のまま忠実に記録したものです。
{{< /callout >}}

---

## 6.1 記述規則定義言語

記述規則定義言語（OWL-DSP）のオントロジー記述を以下に示します。

```turtle
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix dct: <http://purl.org/dc/terms/> .
@prefix skos: <http://www.w3.org/2004/02/skos/core#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix dsp: <http://purl.org/metainfo/terms/dsp#> .
@base <http://purl.org/metainfo/terms/dsp>.

<> a owl:Ontology ;
   rdfs:label "Description Set Profile Definition Language" ;
   rdfs:comment """メタデータの記述規則を表現するためのメタ言語。メタデー
タのレコード記述規則は、OWLのクラスで表現し、各レコードの項目記述規
則をOWLのクラス制約として表現する。メタデータをレコード記述規則クラス
のインスタンスとして記述することで、推論ツールの拡張による整合性検証
を可能にする。このメタ言語では、制約記述を表現するためのプロパティの
ほか、OWLクラス、OWLクラス制約がレコード記述規則、項目記述規則である
ことを明示するためのメタクラスも定義する。""" ;
   dct:created "2010-12-20" ;
   dct:modified "2011-02-13" ;
   owl:versionInfo "ver. 0.30" .
```

### クラス定義

```turtle
dsp:DescriptionTemplate a owl:Class ;
   rdfs:subClassOf owl:Class ;
   rdfs:label "Description Template" ;
   rdfs:comment """レコード記述規則を表すメタクラス。
レコード記述規則DはOWLのクラス（例えばex:CD）として表現し、その記述
規則Dにしたがって書かれたメタデータ（例えばex:MD）は記述規則クラスex:
CDのインスタンスとなる（すなわち、ex:CD a owl:Class. であり、ex:MD a
ex:CD. となる）。
クラスex:CDがレコード記述規則であることを分かりやすくするため、OWLク
ラスの代わりにDescriptionTemplateのインスタンス（つまりex:CD a dsp:
DescriptionTemplate.）として表現してよい。""" ;
   rdfs:subClassOf
      #項目記述規則をサブクラス関係で結ぶ。
      [ a owl:Restriction ;
         owl:onProperty rdfs:subClassOf;
         owl:onClass dsp:StatementTemplate;
         owl:minQualifiedCardinality 0 ] .
```

```turtle
dsp:StatementTemplate a owl:Class ;
   rdfs:subClassOf owl:Restriction ;
   rdfs:label "Statement Template" ;
   rdfs:comment """項目記述規則を表すメタクラス。
項目記述規則SはOWLのクラス制約（例えばex:RS）として表現し、その記述
規則Sを持つレコード記述規則Dのクラス（例えばex:CD）は、ex:RSのサブク
ラスとなる（すなわち、ex:RS a owl:Restriction. であり、ex:CD rdfs:
subClassOf ex:RS. となる）。レコード記述規則D1が項目記述規則S1、S2、
S3を持つならば、それぞれのクラス、クラス制約の関係はex:CD1 rdfs:
subClassOf ex:RS1, ex:RS2, ex:RS3. となる。レコード記述規則D1に従うメ
タデータは、項目記述規則の制約S1、S2、S3をすべて満たすもの（ex:RS1,
ex:RS2, ex:RS3の共通部分）だからである。
クラス制約ex:RSが項目記述規則であることを分かりやすくするため、OWLク
ラス制約の代わりにStatementTemplateのインスタンス（つまりex:RS a dsp:
StatementTemplate.）として表現してよい。""" ;
   rdfs:subClassOf
      #対象となるプロパティを必ず1つ定義する。
      [ a owl:Restriction ;
         owl:onProperty owl:onProperty;
         owl:cardinality 1 ] .
```

### プロパティ定義

```turtle
dsp:valueURIOccurrence a owl:DatatypeProperty ;
   rdfs:label "Value URI Occurrence" ;
   rdfs:comment """レコードを空白ノードとできるかどうかを示す。プロパティ
値がmandatoryならURI必須、optionalなら空白ノード可、disallowedなら常
に空白ノード。このプロパティを持たない場合は、optionalであるのと同等。
""" ;
   rdfs:domain dsp:DescriptionTemplate ;
   rdfs:range [ a owl:DataRange ;
      owl:oneOf("mandatory" "optional" "disallowed")
   ] .
```

```turtle
dsp:inScheme a owl:ObjectProperty ;
   rdfs:label "In Scheme" ;
   rdfs:comment """クラスのメンバーが、目的語で示される語彙（シソーラス
など）の概念の集合で構成されることを表す。たとえば、国立国会図書館件
名標目表（NDLSH）に含まれるそれぞれの件名を、「NDLSH語彙に属する」と
いう制約で表現される匿名クラスのインスタンスと考えると便利な場合があ
る。このプロパティは、このクラス制約を [dsp:inScheme ndlsh: ] と簡易に
表現するために用いる。このプロパティで表現されたクラス制約について、
推論 {ex:CR dsp:inScheme ndlsh: .} => {ex:CR owl:onProperty skos:
inScheme; owl:allValuesFrom ndlsh: .} が成り立つ。
項目記述規則において、dc:subjectの値制約としてNDLSH語彙を指定する場
合、[dsp:StatementTemplate; owl:onProperty dc:subject; owl:onClass
[dsp:inScheme ndlsh:]] という表現ができる。複数語彙が許される場合、
制約に用いるクラスを [owl:unionOf([dsp:inScheme ndlsh:] [dsp:inScheme
bsh:])] のように和集合クラスとする。
DCMI-DSPのvocabularyEncodingSchemeに近い。""" ;
   rdfs:domain rdfs:Class ;
   rdfs:range skos:ConceptScheme .
```

```turtle
dsp:resourceClass a owl:ObjectProperty ;
   rdfs:label "Resource Class" ;
   rdfs:comment """レコード記述規則によって記述したメタデータインスタン
スは、このクラスのメンバーとなることを示す。DCMI-DSPのresourceClassと
同等。""" ;
   rdfs:domain dsp:DescriptionTemplate ;
   rdfs:range rdfs:Class .
```

```turtle
dsp:cardinalityNote a owl:DatatypeProperty ;
   rdfs:label "Cardinality Note" ;
   rdfs:comment """項目記述規則の出現回数制約のうち、「推奨」「あれば必
須」など数値表現できない制約を記述する。このプロパティがある場合、owl
:minCardinalityは1と解釈される。""" ;
   rdfs:domain dsp:StatementTemplate .
```

```turtle
dsp:langTagOccurrence a owl:DatatypeProperty ;
   rdfs:label "Language Tag Occurrence" ;
   rdfs:comment """項目記述規則の値制約がプレーンリテラルの場合、言語タ
グが必須（mandatory）か、任意（optional）か、不可（disallowed）かを示
す。値がプレーンリテラルとならない項目規則で用いた場合はエラー。""" ;
   rdfs:domain dsp:StatementTemplate ;
   rdfs:range [ a owl:DataRange ;
      owl:oneOf("mandatory" "optional" "disallowed")
   ] .
```

```turtle
dsp:perLangMaxCardinality a owl:DatatypeProperty ;
   rdfs:label "Per Language Max Cardinality" ;
   rdfs:comment """項目記述規則のプロパティが、1つの言語タグあたり最大
何回出現できるかを制約する。特にプロパティ値の読みを言語タグを用いて
表現する場合、この値を1と制約することで、プロパティ値文字列と読みが1
対1に対応することを保証する。値がプレーンリテラルとならない項目規則で
用いた場合はエラー。""" ;
   rdfs:domain dsp:StatementTemplate ;
   rdfs:range xsd:nonNegativeInteger .
```

```turtle
dsp:propertyMapping a owl:ObjectProperty ;
   rdfs:label "Property Mapping" ;
   rdfs:comment """項目記述規則のプロパティPを、ダムダウン（単純化）用
に汎用上位プロパティQと関連付ける。通常はP rdfs:subPropertyOf Qp. 関係
を用いてダムダウンを行なうが、この規則においてはQpとは異なるプロパティ
とマッピングしたい場合や、Pは外部で定義された語彙から利用しており勝
手にsubPropertyOf関係を加えられない場合などに用いる。したがって、{
[a dsp:StatementTemplate; onProperty any:P; dsp:propertyMapping ex:Q]
} => {any:P rdfs:subPropertyOf ex:Q} が暗示される。""" ;
   rdfs:domain dsp:StatementTemplate ;
   rdfs:range rdf:Property .
```

---

## 6.3 OWL記述例

6.2.5の簡易DSP記述例をOWL-DSPに変換し、メタデータを加えたRDFは次のようになる。

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
   rdfs:label "国立国会図書館書誌記述" ;
   reg:created "2011-01-15" ;
   reg:creator ex:aRegisteredUserId ;
   reg:version "第1.1版" .

<#MAIN> a dsp:DescriptionTemplate ;
   dsp:valueURIOccurrence "mandatory" ;
   dsp:resourceClass foaf:Document ;
   reg:idField "書誌ID" ;
   reg:resourceNsURI ndlbooks: ;
   rdfs:subClassOf <#MAIN-タイトル>, <#MAIN-著者>, <#MAIN-発行日>, <#MAIN-主題> .

<#MAIN-タイトル> a dsp:StatementTemplate ;
   rdfs:label "タイトル" ;
   owl:onProperty dcterms:title ;
   owl:qualifiedCardinality 1 ;
   owl:onClass <#構造化タイトル> ;
   rdfs:comment "文書の表題" .

<#MAIN-著者> a dsp:StatementTemplate ;
   rdfs:label "著者" ;
   owl:onProperty dcterms:creator ;
   dsp:cardinalityNote "あれば必須" ;
   owl:onClass foaf:Agent ;
   rdfs:comment "文書の作者" .

<#MAIN-発行日> a dsp:StatementTemplate ;
   rdfs:label "発行日" ;
   owl:onProperty dcterms:issued ;
   dsp:cardinalityNote "あれば必須" ;
   owl:maxQualifiedCardinality 1 ;
   owl:onDataRange xsd:date ;
   rdfs:comment "文書の発行日" .

<#MAIN-主題> a dsp:StatementTemplate ;
   rdfs:label "主題" ;
   owl:onProperty dcterms:subject ;
   owl:onClass [owl:unionOf(
      [dsp:inScheme ndlsh:]
      [dsp:inScheme bsh:]
   )] ;
   rdfs:comment "文書の主題" .

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

> **注:** 原文PDFのp.80–81では `rdfs:Lieral` と表記されているが、これは `rdfs:Literal` の誤植と思われる。本ドキュメントでは修正して記載している。
