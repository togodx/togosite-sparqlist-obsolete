# Glycan subsumption （山本・池田）

## Description

- Data sources
    - [GlyCosmos](https://glycosmos.org/data)

- Query
    - Input
        - GlyTouCan ID
    - Output
        - 

## Endpoint

https://ts.glycosmos.org/sparql

## `data`

```sparql
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX glycan: <http://purl.jp/bio/12/glyco/glycan#>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX sbsmpt: <http://www.glycoinfo.org/glyco/owl/relation#>
PREFIX structures: <https://glytoucan.org/Structures/Glycans/>

SELECT DISTINCT ?parent ?child
FROM <http://rdf.glytoucan.org/partner/glycome-db>
FROM <http://rdf.glytoucan.org/partner/bcsdb>
FROM <http://rdf.glytoucan.org/partner/glycoepitope>
FROM <http://rdf.glycosmos.org/glycans/subsumption>
WHERE {
  ?wurcs a ?parent ;
         rdfs:seeAlso ?child ;
         sbsmpt:subsumes* / dcterms:source / glycan:is_from_source / rdfs:seeAlso ?taxonomy .
  VALUES ?taxonomy { <http://identifiers.org/taxonomy/9606> }
}
#GROUP BY ?parent
#ORDER BY ?count
```

## `return`

```javascript
({data}) => {
  const parentIdPrefix = "http://www.glycoinfo.org/glyco/owl/relation#";
  const childIdPrefix = "https://glytoucan.org/Structures/Glycans/";

  let tree = [{
    id: "root",
    label: "root node",
    root: true
  }];
  let chk = {};
  
  data.results.bindings.map(d => {
    if (!chk[d.parent.value]) {
      chk[d.parent.value] = true;
      tree.push({     
        id: d.parent.value.replace(parentIdPrefix, ""),
        label: d.parent.value.replace(parentIdPrefix, "").replace(/_/g, " "),
        leaf: false,
        parent: "root"
      })
    }
    tree.push({
      id: d.child.value.replace(childIdPrefix, ""),
      label: "",
      leaf: true,
      parent: d.parent.value.replace(parentIdPrefix, "")
    })
  });
  
  return tree;
};
```