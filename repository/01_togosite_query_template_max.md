# TogoSite query template 

IDリストも返せるバージョン（テンプレートとしては要仕上げだが、Togoサイトではとりあえず使わない予定なので保留）

## Parameters

* `categoryIds` フォルトは空でも良い。SPARQLクエリに必要な場合はルートノードの ID が初期値。階層がある場合は中間階層のノードの ID も受け付け、指定ノードの一つ下位階層の内訳を返す。階層も無い場合も、指定カテゴリのリストを所得できるようにする。復数のカテゴリを受け取れるようにする（数値の範囲指定の場合は一つで良い）
  * default: 9992
  * example: 9992 (Molecular function), 9998 (Cellular component), 9999 (Biological process), 472 (Membrane)
* `queryIds` デフォルトは空。数える対象の ID リスト。ユーザが ID のリストを指定した場合、全体の内訳の代わりに、ユーザの ID が各内訳に何個ずつ該当するかを返す。空白文字かコンマ区切りのリスト。”TogoID内部ID形式”を受け取る（内部ID形式の方針が変わったのでそれに合わせる）
  * example: Q9NYF8,Q4V339,A6NCE7,A7E2F4,P69849,A6NN73,Q92928,Q5T1J5,P0C7P4,Q6DN03,P09874,Q08211,Q5T4S7,P12270,Q9UPN3,P07814,P53621,P49321,P0C629,Q9BZK8,Q9BY65
* `mode` 内訳の代わりに該当する ID のリストを返す（デフォルトはオフ）idList: リストだけ、objectList: Attributeの入ったリスト（Attribute は下階層ではなく、categoryid で指定したカテゴリ）
  * example: idList, objectList

## `queryArray`

ユーザが指定した ID リストを配列に分割

```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## `categoryArray`

category ID を配列に分割

```javascript
({categoryIds}) => {
  categoryIds = categoryIds.replace(/,/g," ")
  if (categoryIds.match(/[^\s]/)) return categoryIds.split(/\s+/);
  return false;
}
```

## Endpoint

https://integbio.jp/rdf/mirror/uniprot/sparql

## `data`
- メイン SPARQL
  - 内訳返す場合と遺伝子リスト返す場合を handlbars で条件分岐
  - keyword、タンパク質リストでのフィルタリング
  - ?category a up:Concept でタイプで絞り込まないと遅くなる
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX keywords: <http://purl.uniprot.org/keywords/>
{{#if mode}}
SELECT DISTINCT ?uniprot ?category ?label
{{else}}
SELECT ?category ?label (COUNT (DISTINCT ?uniprot) AS ?count) (SAMPLE(?child_category) AS ?child)
{{/if}}
WHERE {
{{#if queryArray}}
  VALUES ?uniprot { {{#each queryArray}} uniprot:{{this}} {{/each}} }
{{/if}}
{{#if categoryArray}}
  {{#if mode}}
  VALUES ?category { {{#each categoryArray}} keywords:{{this}} {{/each}} }    
  {{else}}
  VALUES ?parent { {{#each categoryArray}} keywords:{{this}} {{/each}} }
  {{/if}}
{{/if}}
  ?uniprot a up:Protein ;
           up:organism taxon:9606 ;
           up:proteome ?proteome ;
           up:classifiedWith/rdfs:subClassOf* ?category .
  {{#unless mode}}
  ?category a up:Concept ;
            rdfs:subClassOf ?parent .
  {{/unless}}
  ?category skos:prefLabel ?label .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
  OPTIONAL {
    ?child_category rdfs:subClassOf ?category .
  }
}
{{#unless mode}}
ORDER BY DESC (?count)
{{/unless}}
```

## `return`
- 整形
```javascript
({mode, data})=>{
  const idVarName = "uniprot";
  const idPrfix = "http://purl.uniprot.org/uniprot/";
  const categoryPrefix = "http://purl.uniprot.org/keywords/";
  if (mode == "objectList") return data.results.bindings.map(d=>{
    return {
      id: d[idVarName].value.replace(idPrfix, ""), 
      attribute: {
        categoryId: d.category.value.replace(categoryPrefix, ""), 
        uri: d.category.value,
        label : d.label.value
      }
    }
  });
  if (mode == "idList") return Array.from(new Set(data.results.bindings.map(d=>d[idVarName].value.replace(idPrfix, "")))); // unique

  return data.results.bindings.map(d=>{ 
    return {
      categoryId: d.category.value.replace(categoryPrefix, ""), 
      label: d.label.value,
      count: Number(d.count.value),
      hasChild: Boolean(d.child)
    };
  });	
}
```