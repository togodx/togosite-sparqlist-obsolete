# uniprot interact proteins (IntAct)（守屋）

- 相互作用相手数の内訳

## Description

- Data sources
    - The number of interacting proteins from UniProt
    - This item based on the data of March, 2021 of UniProt (human only).
- Query
    - Input
        - UniProt ID, Number of interacting proteins
    - Output
        - The number of interacting proteins from UniProt
        - If a UniProt ID is entered, it returns the number of interacting proteins

## Parameters

* `categoryIds` (type: interact proteins range)
  * example: 20-
* `queryIds` (type: uniprot)
  * example: Q9NYF8,Q4V339,A6NCE7,A7E2F4,P69849,A6NN73,Q92928,Q5T1J5,P0C7P4,Q6DN03,P09874,Q08211,Q5T4S7,P12270,Q9UPN3,P07814,P53621,P49321,P0C629,Q9BZK8,Q9BY65
* `mode`
  * example: idList, objectList

## `queryArray`
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

## `data`
- メイン SPARQL
  - 内訳返す場合とタンパク質リスト返す場合を handlebars で条件分岐
  - 相互作用相手数、タンパク質リストでフィルタリング
  - セルフ interaction を別に取得して UNION
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
{{#if mode}}
SELECT DISTINCT ?uniprot ?target_num
{{else}} 
SELECT ?target_num ?label (COUNT(DISTINCT ?uniprot) AS ?count)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
  {
    SELECT ?uniprot (COUNT (DISTINCT ?target) - 1 AS ?target_num)
    WHERE {
{{#if queryArray}}
      VALUES ?uniprot { {{#each queryArray}} uniprot:{{this}} {{/each}} }
{{/if}}
      {
        ?uniprot a up:Protein ;
                 up:organism taxon:9606 ;
                 up:proteome ?proteome ;
                 up:interaction/^up:interaction ?target .
        FILTER (?target != ?uniprot)                   # filter own
        FILTER(REGEX(STR(?proteome), "UP000005640"))
      } UNION {                                        # union self-interaction ?uniprot-?target list
        SELECT DISTINCT ?uniprot ?target
        WHERE {
          {
            SELECT ?uniprot ?interaction (COUNT (DISTINCT ?target) AS ?count)
            WHERE {
              ?uniprot a up:Protein ;
                      up:organism taxon:9606 ;
                      up:proteome ?proteome ;
                      up:interaction ?interaction .
              ?target up:interaction ?interaction  .
              FILTER(REGEX(STR(?proteome), "UP000005640"))
            }
          }
          FILTER (?count = 1)                           # filter selef-interaction
          BIND (?uniprot AS ?target)
        }
      }
    }
  }
}
{{#unless mode}}
ORDER BY ?target_num
{{/unless}}
```

## `return`
- 上位を合算
```javascript
({mode, categoryIds, data})=>{
  if (mode) {
    const idVarName = "uniprot";
    const idPrefix = "http://purl.uniprot.org/uniprot/";
    // add without-binding-site-data
    // if (withoutTarget.results.bindings[0] && without.results.bindings[0].uniprot) data.results.bindings = data.results.bindings.concat(without.results.bindings);
    let filteredData = [];
    if (categoryIds) {
      // range
      let range = {begin: 0, end: Infinity};
      if (categoryIds.match(/^[\d\.]+-/)) range.begin = Number(categoryIds.match(/^([\d\.]+)-/)[1]);
      if (categoryIds.match(/-[\d\.]+$/)) range.end = Number(categoryIds.match(/-([\d\.]+)$/)[1]);
      for (let d of data.results.bindings) {
        if (range.begin <= Number(d.target_num.value) && Number(d.target_num.value) <= range.end) filteredData.push(d);
      }
    } else filteredData = data.results.bindings;
    if (mode == "objectList") return filteredData.map(d=>{
      return {
        id: d[idVarName].value.replace(idPrefix, ""),
        attribute: {categoryId: d.target_num.value, label: d.target_num.value}
      }
    });
    if (mode == "idList") return filteredData.map(d=>d[idVarName].value.replace(idPrefix, ""));
  }
  // 仮想階層制御
  const limit_1 = 20;
  const limit_2 = 100;
  const bin_2 = 10;
  if (!categoryIds) {
    let res = [];
    for (let d of data.results.bindings) {
      let num = Number(d.target_num.value);
      if (num < limit_1) res.push( { categoryId: d.target_num.value, label: d.target_num.value, count: Number(d.count.value)} );
      else if (num >= limit_1 && (res.length == 0 || res[res.length - 1].label != limit_1 + "-")) res.push( { categoryId: limit_1 + "-", label: limit_1 + "-", count: Number(d.count.value), hasChild: true} );
      else res[res.length - 1].count += Number(d.count.value);
    }
    return res;
  } else if (categoryIds == limit_1 + "-") {
    let res = [];
    for (let d of data.results.bindings) {
      let num = Number(d.target_num.value);
      let start = parseInt(num / bin_2) * bin_2;
      let label = start + "-" + (start + 9);
      if (num < limit_1) continue;
      if (num < limit_2 && res.length <= (num - limit_1) / bin_2) res.push( { categoryId: label, label: label, count: Number(d.count.value), hasChild: true} );
      else if (num >= limit_2 && res[res.length - 1].label != limit_2 + "-") res.push( { categoryId: limit_2 + "-", label: limit_2 + "-", count: Number(d.count.value), hasChild: true} );
      else res[res.length - 1].count += Number(d.count.value);
    }
    return res;
  } else {
    let range = categoryIds.split(/-/);
    let res = [];
    for (let d of data.results.bindings) {
      let num = Number(d.target_num.value);
      if (num < Number(range[0]) || (range[1] && num > Number(range[1]))) continue;
      res.push( { categoryId: d.target_num.value, label:  d.target_num.value, count: Number(d.count.value)} );
    }
    return res;
  }
}
```