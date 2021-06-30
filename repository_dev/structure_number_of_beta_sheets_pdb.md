# PDBエントリをbeta_sheetで分類(ヒトのみ) (フロント開発用)（井手, 守屋）

## Description

- Data sources
    - The number of beta-sheets recorded in the PDB entry.
    - This item based on the data of January 20, 2021 of PDBj. 
        - The latest data can be obtained from the URL below. https://data.pdbj.org/pdbjplus/data/pdb/rdf/
- Query
    - Input
        - Number of beta-sheets, PDB id
    - Output
        - The number of PDB entries included in each beta-sheets number
        - If a PDB id is entered, it returns the beta-sheets value contained in each entry.

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters

* `categoryIds`   -sheet(type:β-sheetの数)
  * example: 5 140-150 -2 300-
* `queryIds` -(type: PDB)
  * example: 6TIW,6E7C,1PFL
* `mode`
  * example: idList, objectList

## `queryArray`
- Filter 用 PDB を配列に
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## `filter`
```javascript
({mode, categoryIds})=>{
  if (!mode || !categoryIds) return "";
  else if (categoryIds.match(/^[^-]+$/)) {
    let filters = categoryIds.split(/,/).map(d=>{ return "?target_num = " + d });
　  return "FILTER( " + filters.join(" || ") + " )";
  } else {
    let filters = [];
    if (categoryIds.match(/^[\d\.]+-/)) filters.push("?target_num >= " + categoryIds.match(/^([\d\.]+)-/)[1]);
    if (categoryIds.match(/-[\d\.]+$/)) filters.push("?target_num <= " + categoryIds.match(/-([\d\.]+)$/)[1]);
    return "FILTER( " + filters.join(" && ") + " )";
  }
}
```

## `withTarget`

```sparql
PREFIX pdbr: <https://rdf.wwpdb.org/pdb/>
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
{{#if mode}}
SELECT DISTINCT ?PDBentry ?target_num
{{else}}
SELECT ?target_num (COUNT(?PDBentry) AS ?count)
{{/if}}
WHERE {
  {
    SELECT DISTINCT (COUNT(?sheet) AS ?target_num) ?PDBentry
    WHERE{
      {{#if queryArray}}
       VALUES ?PDBentry { {{#each queryArray}} pdbr:{{this}} {{/each}} }
      {{/if}}
      ?PDBentry  a pdbo:datablock ;
                 pdbo:has_struct_sheetCategory ?sheet .
      ?sheet pdbo:has_struct_sheet ?sheet_each .
      {
        SELECT DISTINCT ?PDBentry {
          ?PDBentry pdbo:has_entityCategory
                  / pdbo:has_entity
                  / rdfs:seeAlso <http://identifiers.org/taxonomy/9606> .
        }
      }
    }
  }
  {{filter}}
}
order by ?target_num
```

## `zero_check`
```javascript
({categoryIds})=>{
  if (!categoryIds || categoryIds.match(/^0*-\d/) || categoryIds.split(/,/).includes("0")) return true;
  return false;
}
```

## `withoutTarget`
- シートを持たないタンパク質の数
```sparql
PREFIX pdbr: <https://rdf.wwpdb.org/pdb/>
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#> 
{{#if mode}}
SELECT DISTINCT ?PDBentry ?target_num
{{else}}
SELECT (COUNT(DISTINCT ?PDBentry) AS ?count)
{{/if}}
WHERE{
{{#if zero_check}}
  {
    SELECT DISTINCT ?PDBentry ?title 
    WHERE {
      {{#if queryArray}}
      VALUES ?PDBentry { {{#each queryArray}} pdbr:{{this}} {{/each}} }
      {{/if}}
      ?PDBentry  a pdbo:datablock ;
                 dc:title ?title .
      MINUS {?PDBentry pdbo:has_struct_confCategory ?sheet . }
      {
        SELECT DISTINCT ?PDBentry {
          ?PDBentry pdbo:has_entityCategory
                  / pdbo:has_entity
                  / rdfs:seeAlso <http://identifiers.org/taxonomy/9606> .
        }
      }
    }
  }
  BIND ("0" AS ?target_num)
{{/if}}
}
```

## `return`
```javascript
({mode, queryIds, categoryIds, withTarget, withoutTarget})=>{
  // renge
  let range = {begin: 0, end: Infinity};
  let cids = false;
  if (categoryIds) {
    if (categoryIds.match(/^[\d\.]+-/)) range.begin = Number(categoryIds.match(/^([\d\.]+)-/)[1]);
    if (categoryIds.match(/-[\d\.]+$/)) range.end = Number(categoryIds.match(/-([\d\.]+)$/)[1]);
    if (categoryIds.match(/^[^-]+$/)) {
      cids = categoryIds.split(/,/).reduce((obj, a)=>{
        obj[Number(a)] = true;
        return obj;
      }, {});
    }
  }
  
  if (mode) {
    const idVarName = "PDBentry";
    const idPrefix = "https://rdf.wwpdb.org/pdb/";
    // add without-binding-site-data
    if (withoutTarget.results.bindings[0] && withoutTarget.results.bindings[0][idVarName]) withTarget.results.bindings = withTarget.results.bindings.concat(withoutTarget.results.bindings);
    let filteredData = [];
    if (categoryIds) {
      for (let d of withTarget.results.bindings) {
        const num = Number(d.target_num.value);
        if ((!cids && range.begin <= num && num <= range.end)
            || (cids[num])) {
          filteredData.push(d);
        }
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

  if (!queryIds || withoutTarget.results.bindings[0].count.value != 0) {
    withTarget.results.bindings.unshift( {count: {value: withoutTarget.results.bindings[0].count.value}, target_num: {value: "0"}}  ); // カウント 0 を追加
  }
  let value = 0;
  let res = [];
  for (let d of withTarget.results.bindings) {
    const num = Number(d.target_num.value);
    if (value < num) {
      // fill missing value by 0
      for (let emptyValue = value; emptyValue < num; emptyValue++) {
        if ((!queryIds) 
            && ((!categoryIds) || ((!cids && range.begin <= emptyValue && emptyValue <= range.end) || (cids[emptyValue])))) {
        	res.push( { categoryId: emptyValue.toString(), label: emptyValue.toString(), count: 0} );
        }
      }
    }
    value = num + 1;
    if ((!categoryIds)
      || ((!cids && range.begin <= num && num <= range.end) || (cids[num]))) {
      res.push( { categoryId: d.target_num.value, label: d.target_num.value, count: Number(d.count.value)} );
    }
  }       
  return res;
}
```
