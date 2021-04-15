# TogoSite query template

- Parameters
  - categoryId (Optional)
  - queryIds (Required)
- Output
  - [ {categoryId:, label:, count:, (hasChild:) } ]

## Parameters

* `categoryId` デフォルトは空でも良い。SPARQLクエリに必要な場合はルートノードの ID が初期値。階層がある場合は中間階層のノードの ID も受け付け、指定ノードの一つ下位階層の内訳を返す。階層も無くクエリにも必要ないならオプショナルパラメータ
  * default: 9992
  * example: 9992 (Molecular function), 9998 (Cellular component), 9999 (Biological process), 472 (Membrane)
* `queryIds` 必要パラメータ。デフォルトは空。数える対象の ID リスト。ユーザが ID のリストを指定した場合、全体の内訳の代わりに、ユーザの ID が各内訳に何個ずつ該当するかを返す。空白文字かコンマ区切りのリスト。”TogoID内部ID形式”を受け取る
  * example: Q9NYF8,Q4V339,A6NCE7,A7E2F4,P69849,A6NN73,Q92928,Q5T1J5,P0C7P4,Q6DN03,P09874,Q08211,Q5T4S7,P12270,Q9UPN3,P07814,P53621,P49321,P0C629,Q9BZK8,Q9BY65

## `queryArray`
- ユーザが指定した ID リストを配列に分割

```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## Endpoint

https://integbio.jp/rdf/mirror/uniprot/sparql

## `data`
- categoryId があった場合に絞り込み
- queryIds があった場合に絞り込み
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX keywords: <http://purl.uniprot.org/keywords/>
SELECT ?category ?label (COUNT (DISTINCT ?uniprot) AS ?count)
WHERE {
{{#if queryArray}}
  VALUES ?uniprot { {{#each queryArray}} uniprot:{{this}} {{/each}} }
{{/if}}
{{#if categoryId}}
  VALUES ?parent { keywords:{{categoryId}} }
{{/if}}
  ?uniprot a up:Protein ;
           up:organism taxon:9606 ;
           up:proteome ?proteome ;
           up:classifiedWith/rdfs:subClassOf* ?category .
  ?category a up:Concept ;
            rdfs:subClassOf ?parent ;
            skos:prefLabel ?label .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
}
```

## `return`
- 整形
- カテゴリが階層構造で、目的の categortID に下階層がある場合は hasChild: true にする
  - SPARQL で取る場合
    - "OPTIONAL { ?child_category subClassOf ?category . }" で目的カテゴリの子供をオプショナルで取得 (subClassOf は RDF に適した predicate)
    - SELECT 文に "(SAMPLE(?child_category) AS ?child)" で一つだけ入れる
    - 下の JavaScript 部分の return に "hasChild: Boolean(d.child)" で真偽値を追加
    - 参考 SPARQList: uniprot_keywors, chembl_mesh
```javascript
({data})=>{
  return data.results.bindings.map(d=>{ 
    return {
      categoryId: d.category.value.replace("http://purl.uniprot.org/keywords/", ""), 
      label: d.label.value,
      count: Number(d.count.value)
      // , hasChild: Boolead(d.child)   //// または hasChild: true, false で記述
    };
  });	
}
```