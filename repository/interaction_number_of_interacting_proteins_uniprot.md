# uniprot interact human proteins (IntAct)（守屋）

- 相互作用相手数の内訳
  - human protein - human protein のみ
  - endpoint に human しか入っていないため
  - 本家 UniProt には virus protein との相互作用もあるが、RDF が種に依存してるので TogoSite endpoint では相手が取れない

## Description

- Data sources
    - The number of interacting proteins from UniProt
    - This item based on the data of March, 2021 of UniProt (human only).
- Query
    - Input
        - UniProt ID, Number of interacting human proteins
    - Output
        - The number of interacting human proteins from UniProt
        - If a UniProt ID is entered, it returns the number of interacting proteins

## Parameters

* `categoryIds` (type: interact proteins range)
  * example: 3-5 -5 20-
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

## Endpoint
https://integbio.jp/togosite/sparql

## `data`
- メイン SPARQL
  - 内訳返す場合とタンパク質リスト返す場合を handlebars で条件分岐
  - 相互作用相手数、タンパク質リストでフィルタリング
  - セルフ interaction, interaction 無しを別に取得して UNION
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
{{#if mode}}
SELECT DISTINCT ?uniprot ?target_num
{{else}} 
SELECT ?target_num (COUNT(DISTINCT ?uniprot) AS ?count)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
  # union non-self-, self-, non-
  {
    SELECT ?uniprot (COUNT (DISTINCT ?target) AS ?target_num)
    WHERE {
{{#if queryArray}}
      VALUES ?uniprot { {{#each queryArray}} uniprot:{{this}} {{/each}} }
{{/if}}
      { # non-self-interaction
        ?uniprot a up:Protein ;
                 up:organism taxon:9606 ;
                 up:proteome ?proteome ;
                 up:interaction ?interaction .
        ?interaction a up:Non_Self_Interaction ;  # non self
                     ^up:interaction ?target .
        FILTER(REGEX(STR(?proteome), "UP000005640"))
        FILTER (?target != ?uniprot)
      } UNION { # self-interaction
        ?uniprot a up:Protein ;
                 up:organism taxon:9606 ;
                 up:proteome ?proteome ;
                 up:interaction ?interaction .
        ?interaction a up:Self_Interaction ;  # self
                     ^up:interaction ?target .
        FILTER(REGEX(STR(?proteome), "UP000005640"))
        FILTER (?target = ?uniprot)
      } 
    }
  } UNION { # non-interaction
    SELECT ?uniprot ?target_num
    WHERE {
{{#if queryArray}}
      VALUES ?uniprot { {{#each queryArray}} uniprot:{{this}} {{/each}} }
{{/if}}
      ?uniprot a up:Protein ;
               up:organism taxon:9606 ;
               up:proteome ?proteome .
      FILTER(REGEX(STR(?proteome), "UP000005640"))    
      MINUS { ?uniprot up:interaction [] . }   
      BIND (0 AS ?target_num)
    }
  }
  {{filter}}
}
{{#unless mode}}
ORDER BY ?target_num
{{/unless}}
```

## `results`

```javascript
({mode, queryIds, categoryIds, data})=>{
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
    const idVarName = "uniprot";
    const idPrefix = "http://purl.uniprot.org/uniprot/";
    let filteredData = [];
    if (categoryIds) {
      for (let d of data.results.bindings) {
        const num = Number(d.target_num.value);
        if ((!cids && range.begin <= num && num <= range.end)
            || (cids[num])) {
          filteredData.push(d);
        }
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

  let value = 0;
  let res = [];
  for (let d of data.results.bindings) {
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
