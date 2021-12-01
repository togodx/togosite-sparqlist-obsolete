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
select distinct  ?parent
from <http://ddbj.nig.ac.jp/ontologies/taxonomy>
where {
?taxlevel1 a ddbjtax:Taxon ;
rdfs:subClassOf  taxon:33090 .
  BIND(IRI(REPLACE(STR(?taxlevel1), "http://identifiers.org/taxonomy/","")) AS ?parent)
}

```

## `list1`
```javascript
({Top}) => {
  
 let tree1 = [
  ];
  
 Top.results.bindings.map(d =>
    tree1.push(
      d.parent.value,
  )
      );

  return tree1;
}

```

## `TaxIDList`
```sparql
prefix ddbjtax: <http://ddbj.nig.ac.jp/ontologies/taxonomy/>
prefix dct: <http://purl.org/dc/terms/>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
prefix taxon: <http://identifiers.org/taxonomy/>
select distinct  ?taxlevel2
from <http://ddbj.nig.ac.jp/ontologies/taxonomy>
where {
  VALUES ?taxlevel1 { {{#each list1}} tree:{{this}} {{/each}} }
?taxlevel2 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel1 .
}
```
## `list2`
```javascript
({Top, TaxIDList}) => {
  
 let tree2 = [
  ];
  
 Top.results.bindings.map(d =>
    tree2.push(
      d.taxlevel1.value,
  )
      );
  TaxIDList.results.bindings.map(d =>
    tree2.push(
      d.taxlevel2.value,
  )
      );

  return tree2;
}

```


