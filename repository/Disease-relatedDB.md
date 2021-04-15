# MONDOにおける、関連付けされているDBのカウント(高月)
- 階層はありません
- Parameters
  - queryIds (Required)
  - Output
  - [ {categoryId:, label:, count: } ]
  
## Parameters
* `categoryIds` 必須パラメータ。デフォルトは空。数える対象の ID リスト。
   ユーザが ID のリストを指定した場合、全体の内訳の代わりに、ユーザの ID が各内訳に何個ずつ該当するかを返す。
   空白文字かコンマ区切りのリスト。（TogoID内部ID形式）を受け取る
  * example:DOID,OMIM,MESH
* `queryIds` 必須パラメータ。デフォルトは空。数える対象の ID リスト。
   ユーザが ID のリストを指定した場合、全体の内訳の代わりに、ユーザの ID が各内訳に何個ずつ該当するかを返す。
   空白文字かコンマ区切りのリスト。（TogoID内部ID形式）を受け取る
  * example:0007947,0030049,0007132
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

https://integbio.jp/rdf/mirror/bioportal/sparql

## `data`
- categoryIds があった場合に絞り込み
- queryIds があった場合に絞り込み

```sparql
PREFIX mondo: <http://purl.obolibrary.org/obo/MONDO_>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX oboinowl: <http://www.geneontology.org/formats/oboInOwl#>

{{#if mode}}
 SELECT DISTINCT ?mondo ?name
{{else}}
SELECT DISTINCT ?name (count (?name) AS ?count)
{{/if}}       
FROM <http://integbio.jp/rdf/mirror/bioportal/mondo>
WHERE {
{{#if queryArray}}
  VALUES ?mondo { {{#each queryArray}} mondo:{{this}} {{/each}} }
{{/if}}
{{#if categoryArray}}
  VALUES ?category { {{#each categoryArray}} "{{this}}" {{/each}} } 
  ?mondo oboinowl:hasDbXref ?id .
  FILTER(REGEX(?id,?category))
  BIND (strbefore(?id, ":") AS ?name).
{{else}}
   ?mondo oboinowl:hasDbXref ?id .
   BIND (strbefore(?id, ":") AS ?name)  
{{/if}}
   FILTER(!STRSTARTS(?id, "http"))
 }
{{#if mode}}
ORDER BY DESC(?mondo)
{{else if queryArray}}
GROUP BY ?name ORDER BY DESC(?count)
{{else}}
GROUP BY ?name HAVING (count (?name) > 580) ORDER BY DESC(?count)
{{/if}} 

```
## `return`
- 整形
```javascript
({ data, mode }) => {
  if (mode === "idList") {
    return Array.from(new Set(
      data.results.bindings.map((d) =>
        d.mondo.value.replace("http://purl.obolibrary.org/obo/MONDO_", "")
      )
    ));
  } else if (mode === "objectList") {
    return data.results.bindings.map((d) => ({
      id: d.mondo.value.replace("http://purl.obolibrary.org/obo/MONDO_", ""),
      attribute: {
        categoryId: d.name.value,
        label: d.name.value
      }
    }));
  } else {
    return data.results.bindings.map((d) => ({
      categoryId: d.name.value,
      label: d.name.value,
      count: Number(d.count.value)
    }));
  }
};
```

## MEMO
-Author
 - Takatsuki