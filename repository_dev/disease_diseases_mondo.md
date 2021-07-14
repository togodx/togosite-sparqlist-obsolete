# Diseaseカテゴリフィルタ(Mondo階層利用)(三橋)

## Description

- Data sources
    -  [Mondo Disease Ontology (Mondo) ](https://mondo.monarchinitiative.org/) 
- Query
    - Input
        - Mondo id
    - Output
        -  [Disease and disorder (MONDO_0000001)](https://monarchinitiative.org/disease/MONDO:0000001) and its subcategories of Mondo

## Parameters

* `categoryIds` 指定したMondoノードのリストの下位階層のノード数を返す。
  * default:0000001
  * example: 0000001 (disease and disorder), 0004992, 0005071  (cancer, nervous system disorder),  Search MONDO at https://monarchinitiative.org/
* `queryIds` 数える対象(MONDO_ ID)リスト。ここで指定されたID群が各内訳に何個ずつ該当するかを返す。
  * example: 0008903,0002691,0005260    (liver cancer, lung cancer,autism (disease))
* `mode` 必須パラメータ。内訳の代わりに該当する ID のリストを返す（デフォルトはオフ）idList: リストだけ、objectList: Attributeの入ったリスト（Attribute は下階層ではなく、categoryid で指定したカテゴリ）
  * example: idList, objectList

 ## testURL
  - [default](https://integbio.jp/togosite/sparqlist/api/disease_mondo_filter?categoryIds=0000001&queryIds=&mode=)
  - [catgoryId+queryId+idList](https://integbio.jp/togosite/sparqlist/api/disease_mondo_filter?categoryIds=0000001&queryIds=0008903%2C0002691%2C0005260&mode=idList)
  - [catgoryId+queryId+objectList](https://integbio.jp/togosite/sparqlist/api/disease_mondo_filter?categoryIds=0000001&queryIds=0008903%2C0002691%2C0005260&mode=objectList)

## `queryArray`
- ユーザが指定した ID リストを配列に分割

```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
   if (queryIds.match(/[^\s]/))  return queryIds.split(/\s+/).map( queryId => "MONDO_" + queryId );
  return false;
}
```

## `categoryArray`
- ユーザが指定したカテゴリ ID リストを配列に分割

```javascript
({categoryIds}) => {
  categoryIds = categoryIds.replace(/,/g," ");
  if (categoryIds.match(/[^\s]/)) return categoryIds.split(/\s+/).map( categoryId => "MONDO_" + categoryId　);
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
PREFIX mondo: <http://purl.obolibrary.org/obo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
{{#if mode}}
SELECT DISTINCT ?mondo ?category ?label
{{else}}
SELECT ?category ?label (COUNT (DISTINCT ?mondo) AS ?count) 
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/mondo>
WHERE {
{{#if queryArray}}
  VALUES ?mondo { {{#each queryArray}} mondo:{{this}} {{/each}} }
{{/if}}
{{#if categoryArray}}
  {{#if mode}}
  VALUES ?category { {{#each categoryArray}} mondo:{{this}} {{/each}} }    
  {{else}}
  VALUES ?parent { {{#each categoryArray}} mondo:{{this}} {{/each}} }
  {{/if}}
{{/if}}
{{#unless  mode}}
  ?category rdfs:subClassOf ?parent.
{{/unless}}
  ?category rdfs:label ?label.
  ?mondo rdfs:subClassOf* ?category.
  ?mondo rdf:type owl:Class.  # ?hp rdfs:subClassOf* ?category が?mondoの値に関係なく、trueになってしまうため追加。次のOPTIONALも同じ意図だがこちらの方が軽いはず。
} 
{{#unless mode}}  
ORDER BY DESC(?count)
{{/unless}}
```

## `return`
- 整形
```javascript
({mode, data})=>{
  const idVarName = "mondo";
  const idPrfix = "http://purl.obolibrary.org/obo/MONDO_";
  const categoryPrefix = "http://purl.obolibrary.org/obo/MONDO_";
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
      hasChild: (Number(d.count.value) > 1 ? true : false)
    };
  });	
}
```