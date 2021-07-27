# 日本の難病疾患のカテゴリフィルター（NANDO階層利用）

- Parameters
  - categoryIds (Optional): NANDO_IDを１つ以上指定。空白文字かコンマ区切りのリスト
   * default: 0000001
  - queryIds (Optional): NANDO_IDを１つ以上指定。空白文字かコンマ区切りのリスト
  一カテゴリーは指定難病か小児慢性疾患かで分類されている
- Output
  - [ {categoryId:, label:, count: } ]
  
## Description

- Data sources
    - Nanbyo Disease Ontology (NANDO):[http://nanbyodata.jp/ontology/nando](http://nanbyodata.jp/ontology/nando)
- Input/Output
     -  Input
        - NANDO ID
    - Output
        - NANDO category
- Supplementary information

## Parameters

* `categoryIds` 指定したNANDOノードのリストの下位階層のノード数を返す。
  * default: 0000001
  * example: 1000001 2000001
* `queryIds` 数える対象(NANDO_ ID)リスト。ここで指定されたID群が各内訳に何個ずつ該当するかを返す。
  * example: 1200194 1200196 1200625 1200815 1200915 2200014 2200116 2200204 2100044 2200507 2200714 2200901
* `mode` 必須パラメータ。内訳の代わりに該当する ID のリストを返す（デフォルトはオフ）idList: リストだけ、objectList: Attributeの入ったリスト（Attribute は下階層ではなく、categoryid で指定したカテゴリ）
    * example: idList, objectList
    
## `queryArray`
- ユーザが指定した ID リストを配列に分割

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
  if (categoryIds.match(/[^\s]/)) return  categoryIds.split(/\s+/);
  return false;
}
```

## Endpoint

https://integbio.jp/togosite/sparql

## `data`
- categoryId があった場合に絞り込み
- queryIds があった場合に絞り込み
```sparql

PREFIX nando: <http://nanbyodata.jp/ontology/NANDO_>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

{{#if mode}}
SELECT DISTINCT ?nando ?category STR(?label) AS ?nando_label
{{else}}
SELECT ?category STR(?label) AS ?nando_label (COUNT (DISTINCT ?nando) AS ?count) (SAMPLE(?child_category) AS ?child)
{{/if}}
WHERE {
{{#if queryArray}}
  VALUES ?nando { {{#each queryArray}} nando:{{this}} {{/each}} }
{{/if}}
{{#if categoryArray}}
  {{#if mode}}
  VALUES ?category { {{#each categoryArray}} nando:{{this}} {{/each}} }    
  {{else}}
  VALUES ?parent { {{#each categoryArray}} nando:{{this}} {{/each}} }
  {{/if}}
{{/if}}
 GRAPH <http://rdf.integbio.jp/dataset/togosite/nando> { 
 {{#unless  mode}}
    ?category rdfs:subClassOf ?parent.
 {{/unless}}
    ?category rdfs:label ?label.
    FILTER (lang(?label) = "en")
    ?nando rdfs:subClassOf* ?category.
  }
  OPTIONAL {
    ?child_category rdfs:subClassOf ?category .
  }
} 
{{#unless mode}}  
  ORDER BY DESC(?count)
{{/unless}}
```
## `return`

```javascript
({ data, mode }) => {
  if (mode === "idList") {
    return Array.from(new Set(
      data.results.bindings.map((d) =>
        d.nando.value.replace("http://nanbyodata.jp/ontology/NANDO_", "")
      )
    ));
  } else if (mode === "objectList") {
    return data.results.bindings.map((d) => ({
      id: d.nando.value.replace("http://nanbyodata.jp/ontology/NANDO_", ""),
      attribute: {
        categoryId: d.category.value.replace("http://nanbyodata.jp/ontology/NANDO_", ""),
        uri: d.category.value,
      label: d.nando_label.value
      }
    }));
  } else {
    return data.results.bindings.map((d) => ({
      categoryId: d.category.value
        .replace("http://nanbyodata.jp/ontology/NANDO_", ""),
      label: d.nando_label.value,
      count: Number(d.count.value),
      hasChild: Boolean(d.child)
    }));
  }
};
```


## MEMO
-Author
 - Takatsuki