# Glycan mass （山本・池田）

## Description

- Data sources
    - [GlyCosmos](https://glycosmos.org/data)

- Query
    - Input
        - GlyTouCan ID
    - Output
        - Mass range

## Endpoint

https://ts.glycosmos.org/sparql

## Parameters
* `categoryIds` (type: Da range)
  * example: 200-1000
* `queryIds` (type: GlyTouCan ID)
  * example: G14606UO,G03652TR,G59522PY,G90484GL
* `mode` (type: string)
  * example: idList, objectList

## `input_queries`
```javascript
({ queryIds }) => {
  queryIds = queryIds.replace(/,/g, " ");
  if (queryIds.match(/[^\s]/)) {
    return queryIds.split(/\s+/);
  } else {
    return false;
  }
};
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
  if (range.begin == 4000 && range.end == false) return 100;
  else if (range.begin && range.end && !range.begin < 10000) {
    return (range.end - range.begin) / 10   
  }
  return 200;
}
```

## `data`

```sparql
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX info: <http://rdf.glycoinfo.org/glycan/>
PREFIX glycan: <http://purl.jp/bio/12/glyco/glycan#>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX mass: <https://glycoinfo.gitlab.io/wurcsframework/org/glycoinfo/wurcsframework/1.0.1/wurcsframework-1.0.1.jar#>
PREFIX sbsmpt: <http://www.glycoinfo.org/glyco/owl/relation#>
PREFIX glyconavi: <http://glyconavi.org/owl#>
PREFIX struct: <https://glytoucan.org/Structures/Glycans/>

{{#if mode}}
SELECT DISTINCT ?mass ?glytoucan
{{else}}
SELECT ?mass_2 ?label (COUNT(?glytoucan) as ?count) 
{{/if}}
WHERE {
  {{#if input_queries}}
  VALUES ?glytoucan { {{#each input_queries}} struct:{{this}} {{/each}} }
  {{/if}}
  []
    mass:WURCSMassCalculator ?mass ;
    rdfs:seeAlso ?glytoucan ;
    dcterms:source / glycan:is_from_source / rdfs:seeAlso ?taxnomy .
  VALUES ?taxnomy { <http://identifiers.org/taxonomy/9606> }
  {{#if range}}
      FILTER (
        {{#if range.begin}} 
          ?mass > {{range.begin}}
          {{#if range.end}} && {{/if}}
        {{/if}}
        {{#if range.end}}
          ?mass <= {{range.end}}
        {{/if}}
      )
  {{/if}}
{{#if mode}}
}
{{else}}
  BIND ( CEIL(xsd:float(?mass) / {{bin}}) - 1 AS ?mass_2)
  BIND ( CONCAT (?mass_2 * {{bin}}, "-"  , (?mass_2 + 1) * {{bin}}) AS ?label)
}
GROUP BY ?mass_2 ?label
ORDER BY ?mass_2
{{/if}}
```

## `return`
```javascript
({categoryIds, mode, bin, range, data})=>{
  if (mode) {
    const idVarName = "glytoucan";
    const idPrefix = "https://glytoucan.org/Structures/Glycans/";
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
      if (num < 4000) res.push( { categoryId: d.label.value, label: d.label.value + " Da", count: Number(d.count.value), hasChild: true} );
  //    else if (num < 1000 && num % 100 == 0) res.push( { categoryId: num + "-" + (num + 100), label: num + "-" + (num + 100) + " kDa", count: Number(d.count.value)} );
      else if (num == 4000) res.push( { categoryId: "4000-", label: "4000- Da", count: Number(d.count.value), hasChild: true} );
      else res[res.length - 1].count += Number(d.count.value);
    }
    return res;
  } else if (range.begin == 4000 && range.end == false) {
    let res = [];
    for (let d of data.results.bindings) {
      let num = parseInt(d.mass_2.value) * bin;
      if (num < 10000) res.push( { categoryId: d.label.value, label: d.label.value + " Da", count: Number(d.count.value)} );
      else if (num == 10000) res.push( { categoryId: "10000-", label: "10000- Da", count: Number(d.count.value)} );
      else res[res.length - 1].count += Number(d.count.value);
    }
    return res;
  } else {
    return data.results.bindings.map(d=>{
      return {
        categoryId: d.label.value, 
        label: d.label.value + " Da",
        count: d.count.value
      };
    });
  }
}
```
