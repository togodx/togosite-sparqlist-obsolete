# knapsack  (信定)
- 生物種ーknapsack compoundの分類 
- kingdom-family-oraganism-compound

## Endpoint
https://mb2.ddbj.nig.ac.jp/sparql

## `leaf`
```sparql
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
prefix metabo: <http://ddbj.nig.ac.jp/ontolofies/metabobank/>
prefix dc: <http://purl.org/dc/elements/1.1/>
select distinct   ?tax_id ?compound_id ?compound_label 
from <http://mb-wiki.nig.ac.jp/resource>
where {
?s  a metabo:KNApSAcKCoreAnnotations ;
rdfs:seeAlso ?taxonomy .
BIND (IRI(strbefore(str(?s ), "/organism") ) AS ?molecule)
BIND (strafter(str(?taxonomy), "http://identifiers.org/taxonomy/") AS ?tax_id)
?molecule a metabo:KNApSAcKCoreRecord ;
dc:identifier ?compound_id ;
rdfs:label ?compound_label .
}
limit 10
```
## `graph_a`
```sparql
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
prefix metabo: <http://ddbj.nig.ac.jp/ontolofies/metabobank/>
select distinct   ?tax_id  ?organism  ?family 
from <http://mb-wiki.nig.ac.jp/resource>
where {
?s  a metabo:KNApSAcKCoreAnnotations ;
rdfs:seeAlso ?taxonomy ;
metabo:family ?family ;
metabo:organism ?organism .
BIND (strafter(str(?taxonomy), "http://identifiers.org/taxonomy/") AS ?tax_id)
}
limit 10
```
## `graph_b`
```sparql
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
prefix metabo: <http://ddbj.nig.ac.jp/ontolofies/metabobank/>
select distinct    ?family ?kingdom
from <http://mb-wiki.nig.ac.jp/resource>
where {
?s  a metabo:KNApSAcKCoreAnnotations ;
rdfs:seeAlso ?taxonomy ;
metabo:family ?family ;
metabo:kingdom ?kingdom  .
}
limit 10
```
## `return`
```javascript
({ leaf, graph_a, graph_b}) => {
  
 let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];
  
 leaf.results.bindings.map(d => {
    tree.push({
      id: d.compound_id.value,
      label: d.compound_label.value,
      leaf: true,
      parent: d.tax_id.value
    })
      });
  graph_a.results.bindings.map(d => {
    tree.push({
      id: d.tax_id.value,
      label: d.organism.value,
      parent: d.family.value
    })
  }) ;
graph_b.results.bindings.map(d => {
    tree.push({
      id: d.family.value,
      label: d.family.value,
      parent: d.kingdom.value
    })
  }) ;

 const top = [
  graph_b.results.bindings.map(d => {
    { id: d.kingdom.value,
      label: d.kingdom.value,
      parent: "root"
    }}
  ) ] ;
  
 tree.push(d => {
   const topData = top.filter((element, index, self) => 
                            self.findIndex(e => 
                                           e.id === element.id &&
                                           e.label === element.label
                                          ) === index
                            )
           } );
  
  return tree;
}

```
