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

## `data`

```sparql
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX glycan: <http://purl.jp/bio/12/glyco/glycan#>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX mass: <https://glycoinfo.gitlab.io/wurcsframework/org/glycoinfo/wurcsframework/1.0.1/wurcsframework-1.0.1.jar#>
PREFIX sbsmpt: <http://www.glycoinfo.org/glyco/owl/relation#>
PREFIX glyconavi: <http://glyconavi.org/owl#>
PREFIX struct: <https://glytoucan.org/Structures/Glycans/>

SELECT DISTINCT ?mass ?glytoucan
WHERE {
  ?s mass:WURCSMassCalculator ?mass ;
     rdfs:seeAlso ?glytoucan ;
     sbsmpt:subsumes* / dcterms:source / glycan:is_from_source / rdfs:seeAlso ?taxonomy .
  VALUES ?taxonomy { <http://identifiers.org/taxonomy/9606> }
}
```

## `return`
```javascript
({data}) => {
  const idPrefix = "https://glytoucan.org/Structures/Glycans/";
  
  let tree = [];
  data.results.bindings.map(d => {
    const num = parseInt(Number(d.mass.value) / 200) * 200;
    const bin_id = num + "-" + (num + 200);
    tree.push({
      id: d.glytoucan.value.replace(idPrefix, ""),
      label: "",
      value: Number(d.mass.value),
      binId: bin_id,
      binLabel: bin_id + " Da"
    })
  })
  
  return tree;
}
```
