# Test用 (信定)

## Endpoint
https://mb2.ddbj.nig.ac.jp/sparql


## `Top`
- Taxonomy ID: 33090 (Viridiplantae)
```sparql
prefix ddbjtax: <http://ddbj.nig.ac.jp/ontologies/taxonomy/>
prefix dct: <http://purl.org/dc/terms/>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
prefix taxon: <http://identifiers.org/taxonomy/>
select distinct  ?tax1 ?tax2
from <http://ddbj.nig.ac.jp/ontologies/taxonomy>
where {
?taxlevel1 a ddbjtax:Taxon ;
rdfs:subClassOf  taxon:33090 .
?taxlevel2 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel1 .
  BIND(IRI(REPLACE(STR(?taxlevel1), "http://identifiers.org/taxonomy/","")) AS ?tax1)
  BIND(IRI(REPLACE(STR(?taxlevel2), "http://identifiers.org/taxonomy/","")) AS ?tax2)
}

```

## `list`
```javascript
({Top}) => {
  
 let tree = [
  ];
  
 Top.results.bindings.map(d =>
    tree.push(
      d.tax1.value,
      d.tax2.value,
  )
      );

  return tree;
}

```

##`removeDuplicate`
```javascript
({list}) => {

 let tree2 = [
  ];
  
 list.results.bindings.map(d =>
   tree2.push(
      d.tree.value,
  )
      );
   return tree2 ;
}
  
```

