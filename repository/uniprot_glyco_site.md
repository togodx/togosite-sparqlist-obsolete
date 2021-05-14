# uniprot glyco site（守屋）

- タンパク質糖鎖結合サイト数の内訳

## Parameters

* `categoryIds` (type: glyco site range)
  * example: 20-
* `queryIds` (type: uniprot)
  * example: Q9NYF8,Q4V339,A6NCE7,A7E2F4,P69849,A6NN73,Q92928,Q5T1J5,P0C7P4,Q6DN03,P09874,Q08211,Q5T4S7,P12270,Q9UPN3,P07814,P53621,P49321,P0C629,Q9BZK8,Q9BY65
* `mode`
  * example: idList, objectList

## `query_array`
- Query UniProt ID を配列に
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## Endpoint
https://integbio.jp/togosite/sparql

## `withTarget`
- メイン SPARQL
  - 内訳返す場合とタンパク質リスト返す場合を handlbars で条件分岐
  - 糖鎖結合サイト数、タンパク質リストでフィルタリング
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX core: <http://purl.uniprot.org/core/>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
{{#if mode}}
SELECT DISTINCT ?uniprot ?target_num
{{else}} 
SELECT ?target_num (COUNT (DISTINCT ?uniprot) AS ?count)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
  {
    SELECT DISTINCT ?uniprot (COUNT (DISTINCT ?annotation) AS ?target_num)
    WHERE
    {
{{#if query_array}}
      VALUES ?uniprot { {{#each query_array}} up:{{this}} {{/each}} }
{{/if}}
      ?uniprot a core:Protein ;
               core:organism taxon:9606 ;
               core:proteome ?proteome ;
               core:annotation ?annotation .
      ?annotation a core:Glycosylation_Annotation .
      FILTER(REGEX(STR(?proteome), "UP000005640"))
    }
  }
}
{{#unless mode}}
ORDER BY ?target_num
{{/unless}}
```

## `zero_check`
```javascript
({categoryIds})=>{
  if (!categoryIds || categoryIds.match(/^0*-\d/)) return true;
  return false;
}
```

## `withoutTarget`
- リン酸化サイトの無いタンパク質取得
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX core: <http://purl.uniprot.org/core/>
PREFIX up: <http://purl.uniprot.org/uniprot/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
{{#if mode}}
SELECT DISTINCT ?uniprot ?target_num
{{else}} 
SELECT (COUNT (DISTINCT ?uniprot) AS ?count)
{{/if}}
WHERE {
{{#if zero_check}}
{{#if query_array}}
      VALUES ?uniprot { {{#each query_array}} up:{{this}} {{/each}} }
{{/if}}
  ?uniprot a core:Protein ;
           core:organism taxon:9606 ;
           core:proteome ?proteome .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
  MINUS {
    ?uniprot core:annotation ?annotation .
    ?annotation a core:Glycosylation_Annotation .
  }
  {{#if mode}}
  BIND("0" AS ?target_num)
  {{/if}}
{{/if}}
}
```

## `return`
- 上位を合算
```javascript
({mode, queryIds, categoryIds, withTarget, withoutTarget})=>{
  if (mode) {
    const idVarName = "uniprot";
    const idPrefix = "http://purl.uniprot.org/uniprot/";
    // add without-binding-site-data
    if (withoutTarget.results.bindings[0] && withoutTarget.results.bindings[0][idVarName]) withTarget.results.bindings = withTarget.results.bindings.concat(withoutTarget.results.bindings);
    let filteredData = [];
    if (categoryIds) {
      // range
      let range = {begin: 0, end: Infinity};
      if (categoryIds.match(/^[\d\.]+-/)) range.begin = Number(categoryIds.match(/^([\d\.]+)-/)[1]);
      if (categoryIds.match(/-[\d\.]+$/)) range.end = Number(categoryIds.match(/-([\d\.]+)$/)[1]);
      for (let d of withTarget.results.bindings) {
        if (range.begin <= Number(d.target_num.value) && Number(d.target_num.value) <= range.end) filteredData.push(d);
      }
    } else filteredData = withTarget.results.bindings;
    if (mode == "objectList") return filteredData.map(d=>{
      return {
        id: d[idVarName].value.replace(idPrefix, ""),
        attribute: {categoryId: d.target_num.value, label: d.target_num.value}
      }
    });
    if (mode == "idList") return filteredData.map(d=>d[idVarName].value.replace(idPrefix, ""));
  }
  // 仮想階層制御
  if (!queryIds || withoutTarget.results.bindings[0].count.value != 0) {
    withTarget.results.bindings.unshift( {count: {value: withoutTarget.results.bindings[0].count.value}, target_num: {value: "0"}}  ); // カウント 0 を追加
  }
  let value = 0;
  let res = [];
  for (let d of withTarget.results.bindings) {
    const num = Number(d.target_num.value);
    if (value < num) {
      for (let emptyValue = value; emptyValue < num; emptyValue++) {
        res.push( { categoryId: emptyValue.toString(), label: emptyValue.toString(), count: 0} );
      }
    }
    value = num + 1;
    res.push( { categoryId: d.target_num.value, label: d.target_num.value, count: Number(d.count.value)} );
  }
  return res;
}
```