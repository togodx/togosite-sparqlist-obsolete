# Diseaseカテゴリフィルタ(HPO階層利用)(三橋)

## Description

- Data sources
    -  [Human Phenotype Ontology (HPO)](https://hpo.jax.org/app/) 
- Query
    - Input
        - HPO id
    - Output
        -  [Phenotypic abnormality (HP:0000118)](https://hpo.jax.org/app/browse/term/HP:0000118)  and its subcategories of HPO

## Parameters

* `categoryIds` 指定したHPOノードのリストの下位階層のノード数を返す。
  * default: 0000118
  * example: 0000118 (Phenotypic abnormality), 0033127, 0000707  (Abnormality of the musculoskeletal system, Abnormality of the nervous system) Search HPO at https://hpo.jax.org/
* `queryIds` 数える対象(HPO_ ID)リスト。ここで指定されたID群が各内訳に何個ずつ該当するかを返す。
  * example: 0003418, 0002027,0001945
* `mode` 必須パラメータ。内訳の代わりに該当する ID のリストを返す（デフォルトはオフ）idList: リストだけ、objectList: Attributeの入ったリスト（Attribute は下階層ではなく、categoryid で指定したカテゴリ）
  * example: idList, objectList

 ## testURL
  - [default](https://integbio.jp/togosite/sparqlist/api/disease_hpo_filter?categoryIds=0000118&queryIds=&mode=)
  - [catgoryId+queryId+idList](https://integbio.jp/togosite/sparqlist/api/disease_hpo_filter?categoryIds=0000118%2C0000707&queryIds=0003418%2C0002027%2C0001945&mode=idList)
  - [catgoryId+queryId+objectList](https://integbio.jp/togosite/sparqlist/api/disease_hpo_filter?categoryIds=0000118%2C0000707&queryIds=0003418%2C0002027%2C0001945&mode=objectList)
  - [catgoryId+queryId(存在しないaaaa)+idList](https://integbio.jp/togosite/sparqlist/api/disease_hpo_filter?categoryIds=0000118%2C0000707&queryIds=aaaa&mode=idList)→検索結果がないことを確認

- hasChild 入り
- Authors:
  - 三橋、高月、仲里、藤原

## `queryArray`
- ユーザが指定した ID リストを配列に分割

```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
   if (queryIds.match(/[^\s]/))  return queryIds.split(/\s+/).map( queryId => "HP_" + queryId );
  return false;
}
```

## `categoryArray`
- ユーザが指定したカテゴリ ID リストを配列に分割

```javascript
({categoryIds}) => {
  categoryIds = categoryIds.replace(/,/g," ");
  if (categoryIds.match(/[^\s]/)) return categoryIds.split(/\s+/).map( categoryId => "HP_" + categoryId　);
  return false;
}
```

## Endpoint

https://integbio.jp/togosite/sparql

## `data`
- categoryId があった場合に絞り込み
- queryIds があった場合に絞り込み
```sparql
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX hpo: <http://purl.obolibrary.org/obo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
{{#if mode}}
SELECT DISTINCT ?hp ?category ?label
{{else}}
SELECT ?category ?label (COUNT (DISTINCT ?hp) AS ?count) 
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/hpo>
WHERE {
{{#if queryArray}}
  VALUES ?hp { {{#each queryArray}} hpo:{{this}} {{/each}} }
{{/if}}
{{#if categoryArray}}
  {{#if mode}}
  VALUES ?category { {{#each categoryArray}} hpo:{{this}} {{/each}} }    
  {{else}}
  VALUES ?parent { {{#each categoryArray}} hpo:{{this}} {{/each}} }
  {{/if}}
{{/if}}
{{#unless  mode}}
  ?category rdfs:subClassOf ?parent.
{{/unless}}
  ?category rdfs:label ?label.
  ?hp rdfs:subClassOf* ?category.
  ?hp rdf:type owl:Class.  # ?hp rdfs:subClassOf* ?category が?hpの値に関係なく、trueになってしまうため追加。次のOPTIONALも同じ意図だがこちらの方が軽いはず。
} 
{{#unless mode}}  
ORDER BY DESC(?count)
{{/unless}}
```

## `return`
- 整形
```javascript
({mode, data})=>{
  const idVarName = "hp";
  const idPrfix = "http://purl.obolibrary.org/obo/HP_";
  const categoryPrefix = "http://purl.obolibrary.org/obo/HP_";
  if (mode == "objectList") return data.results.bindings.map(d=>{
    return {
      id: d[idVarName].value.replace(idPrfix, ""), 
      attribute: {
        categoryId: d.category.value.replace(categoryPrefix, ""), 
        uri: d.category.value,
        label : d.label.value.replace(/^Abnormality of /,"").replace(/^the /,"")
      }
    }
  });
  if (mode == "idList") return Array.from(new Set(data.results.bindings.map(d=>d[idVarName].value.replace(idPrfix, "")))); // unique

  return data.results.bindings.map(d=>{ 
    return {
      categoryId: d.category.value.replace(categoryPrefix, ""), 
      label: d.label.value.replace(/^Abnormality of /,"").replace(/the /,""),
      count: Number(d.count.value),
      hasChild: (Number(d.count.value) > 1 ? true : false)
    };
  });	
}
```