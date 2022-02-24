# knapsack organism-compund (信定)
- 生物種ーknapsack compoundの分類 
- kingdom-family-oraganism-compound

## Endpoint
https://mb2.ddbj.nig.ac.jp/sparql

## `main`
```sparql
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
prefix metabo: <http://ddbj.nig.ac.jp/ontolofies/metabobank/>
prefix dc: <http://purl.org/dc/elements/1.1/>
select distinct  ?kingdom  ?family   ?tax_id   ?organism   ?compound_id ?compound_label    
from <http://mb-wiki.nig.ac.jp/resource>
where {
?s  a metabo:KNApSAcKCoreAnnotations ;
rdfs:seeAlso ?taxonomy ;
metabo:family ?family ;
metabo:kingdom ?kingdom ;
metabo:organism ?organism .
BIND (IRI(strbefore(str(?s ), "/organism") ) AS ?molecule)
BIND (strafter(str(?taxonomy), "http://identifiers.org/taxonomy/") AS ?tax_id)
?molecule a metabo:KNApSAcKCoreRecord ;
dc:identifier ?compound_id ;
rdfs:label ?compound_label .
}
limit 30000
```

## `return`
```javascript
({ main}) => {
  
 let tree = [
    {
      id: "root",
      label: "root node",
      root: true
    }
  ];
  
main.results.bindings.map(d => {
    tree.push({
      id: d.compound_id.value,
      label: d.compound_label.value,
      leaf: true,
      parent: d.tax_id.value
    })
      });
main.results.bindings.map(d => {
    tree.push({
      id: d.tax_id.value,
      label: d.organism.value,
      parent: d.family.value
    })
  }) ;
main.results.bindings.map(d => {
    tree.push({
      id: d.family.value,
      label: d.family.value,
      parent: d.kingdom.value
    })
  }) ;

main.results.bindings.map(d => {
    tree.push({
      id: d.kingdom.value,
      label: d.kingdom.value,
      parent: "root"
   })
  }) ;
  
const uniqueTree = Array.from(
  new Map(tree.map((tree2) => [tree2.id, tree2])).values()
)
  
  return uniqueTree;
}

```
