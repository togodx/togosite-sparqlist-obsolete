# Test用 (信定)

## Endpoint
https://mb2.ddbj.nig.ac.jp/sparql


## `TaxIDList`
- Taxonomy ID: 33090 (Viridiplantae)
```sparql
prefix ddbjtax: <http://ddbj.nig.ac.jp/ontologies/taxonomy/>
prefix dct: <http://purl.org/dc/terms/>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
prefix taxon: <http://identifiers.org/taxonomy/>
select distinct  ?taxlevel1 ?taxlevel2
from <http://ddbj.nig.ac.jp/ontologies/taxonomy>
where {
values ?taxlevel0 {taxon:33090} 
?taxlevel1 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel0 .
?taxlevel2 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel1 .
}
limit 1000
```

## `list1`
```javascript
({TaxIDList}) => {
  
 let tree = [
  ];
  
  TaxIDList.results.bindings.map(d =>
    tree.push(
      d.taxlevel1.value,
      d.taxlevel2.value,
  )
      );

  return tree;
}

```
