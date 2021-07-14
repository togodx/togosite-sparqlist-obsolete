# uniprot mass（守屋）

- タンパク質分子量の内訳
  - default: 0-200 kDa まで bin 10kDa, 200- kDa 合算
  - range がある場合 bin = range / 10
    - なので range は 10^^n 刻みが望ましい

## Description

- Data sources
    - [UniProt](https://www.uniprot.org/)

- Query
    - Input
        - UniProt ID
    - Output
        - Molecular mass (kDa) range

## Parameters

* `categoryIds` (type: mass range)
  * example: 30-40 (kDa range)
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

## `range`
- range を抽出
```javascript
({categoryIds})=>{
  categoryIds = categoryIds.replace(/\s/g, "");
  if (categoryIds.match(/\d-/) || categoryIds.match(/-\d/)) {
    let range = {begin: false, end: false};
    if (categoryIds.match(/^[\d\.]+-/)) range.begin = Number(categoryIds.match(/^([\d\.]+)-/)[1]);
    if (categoryIds.match(/-[\d\.]+$/)) range.end = Number(categoryIds.match(/-([\d\.]+)$/)[1]);
    return range;         
  }
  return false;
}
```

## `bin`
- bin を決定
```javascript
({range})=>{
  if (range.begin == 200 && range.end == false) return 100;
  else if (range.begin && range.end && !range.begin < 1000) {
    return (range.end - range.begin) / 10   
  }
  return 10;
}
```

## Endpoint
https://integbio.jp/togosite/sparql

## `data`
- メイン SPARQL
  - 内訳返す場合とタンパク質リスト返す場合を handlbars で条件分岐
  - 質量、タンパク質リストでフィルタリング
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
{{#if mode}}
SELECT DISTINCT ?uniprot ?mass
{{else}}          
SELECT ?mass_2 ?label (COUNT(DISTINCT ?uniprot) AS ?count)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
  {
    SELECT ?uniprot ?mass
    WHERE {
{{#if query_array}}
      VALUES ?uniprot { {{#each query_array}} uniprot:{{this}} {{/each}} }
{{/if}}
      ?uniprot a up:Protein ;
               up:organism taxon:9606 ;
               up:proteome ?proteome ;
               up:sequence/up:mass ?mass .
      FILTER(REGEX(STR(?proteome), "UP000005640"))
{{#if range}}
      FILTER (
        {{#if range.begin}} 
          ?mass > {{range.begin}} * 1000 
          {{#if range.end}} && {{/if}}
        {{/if}}
        {{#if range.end}}
          ?mass <= {{range.end}} * 1000
        {{/if}}
      )
{{/if}}
    }
  }
{{#if mode}}
}
{{else}}
  BIND ( CEIL(xsd:float(?mass) / 1000 / {{bin}}) - 1 AS ?mass_2)
  BIND ( CONCAT (?mass_2 * {{bin}}, "-"  , (?mass_2 + 1) * {{bin}}) AS ?label)
}
ORDER BY ?mass_2
{{/if}}
```

## `return`
- 上位を合算
```javascript
({categoryIds, mode, bin, range, data})=>{
  if (mode) {
    const idVarName = "uniprot";
    const idPrefix = "http://purl.uniprot.org/uniprot/";
    if (mode == "objectList") return data.results.bindings.map(d=>{
      return {
        id: d[idVarName].value.replace(idPrefix, ""),
        attribute: {categoryId: d.mass.value, label: d.mass.value + " Da"}
      }
    });
    if (mode == "idList") return data.results.bindings.map(d=>d[idVarName].value.replace(idPrefix, ""));
  }
  // 仮想階層制御
  if (!categoryIds) {
    let res = [];
    for (let d of data.results.bindings) {
      let num = parseInt(d.mass_2.value) * bin;
      if (num < 200) res.push( { categoryId: d.label.value, label: d.label.value + " kDa", count: Number(d.count.value), hasChild: true} );
  //    else if (num < 1000 && num % 100 == 0) res.push( { categoryId: num + "-" + (num + 100), label: num + "-" + (num + 100) + " kDa", count: Number(d.count.value)} );
      else if (num == 200) res.push( { categoryId: "200-", label: "200- kDa", count: Number(d.count.value), hasChild: true} );
      else res[res.length - 1].count += Number(d.count.value);
    }
    return res;
  } else if (range.begin == 200 && range.end == false) {
    let res = [];
    for (let d of data.results.bindings) {
      let num = parseInt(d.mass_2.value) * bin;
      if (num < 1000) res.push( { categoryId: d.label.value, label: d.label.value + " kDa", count: Number(d.count.value)} );
      else if (num == 1000) res.push( { categoryId: "1000-", label: "1000- kDa", count: Number(d.count.value)} );
      else res[res.length - 1].count += Number(d.count.value);
    }
    return res;
  } else {
    return data.results.bindings.map(d=>{
      return {
        categoryId: d.label.value, 
        label: d.label.value + " kDa",
        count: d.count.value
      };
    });
  }
}
```