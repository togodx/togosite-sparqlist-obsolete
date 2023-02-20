# ?カテゴリフィルタ(HPO階層利用)(高月)

## Description

- Data sources
    -  [Human Phenotype Ontology (HPO)](https://hpo.jax.org/app/) 
- Query
    - Input
        - HPO id
    - Output
        -  [Phenotypic abnormality (HP:0000005)](http://purl.obolibrary.org/obo/HP_0000005)  and its subcategories of HPO

## Endpoint

https://integbio.jp/togosite/sparql

## `data`
```sparql

PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>

SELECT DISTINCT ?hp ?label ?parent SAMPLE(?child) AS ?child
FROM <http://rdf.integbio.jp/dataset/togosite/hpo>
WHERE {
  #root nodeはHP_0000005
  VALUES ?root {  obo:HP_0000005  }    
  ?hp rdfs:subClassOf+ ?root.
  ?hp rdfs:label ?label.
  ?hp rdfs:subClassOf ?parent.
  ?hp rdf:type owl:Class.  
  # HP以外のIDも登録されているため、HPに限定するためにフィルターを追加
  FILTER regex(str(?hp), "http://purl.obolibrary.org/obo/HP_" )
  # 中間ノードの場合は？parentに値が存在し、leafノードの場合は?parentは存在しないのでOPTIONALが必要
  OPTIONAL {
    ?hp ^rdfs:subClassOf ?child.
  }
 } 
 GROUP BY ?hp ?parent ?label
             
```
## `return`

```javascript
({data}) => {
  const idPrefix = "http://purl.obolibrary.org/obo/HP_";
  let tree = [
    {
      id: "0000005",
      label: "Mode of inheritance",
      root: true
    }
  ];
  data.results.bindings.forEach(d => {
    tree.push({
      id: d.hp.value.replace(idPrefix, ""),
      label: d.label.value,
      leaf: (d.child == undefined ? true : false),
      parent: d.parent.value.replace(idPrefix, "")
    });
  });
  return tree;
};
```

## MEMO
-Author
 - Takatsuki
 