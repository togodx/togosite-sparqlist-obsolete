# Test用 (信定)

## Endpoint
https://mb2.ddbj.nig.ac.jp/sparql


## `taxonids`
- Taxonomy ID: 33090 (Viridiplantae)
```sparql
prefix ddbjtax: <http://ddbj.nig.ac.jp/ontologies/taxonomy/>
prefix dct: <http://purl.org/dc/terms/>
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
prefix taxon: <http://identifiers.org/taxonomy/>
select distinct ?tax1 ?tax2 ?tax3 ?tax4 ?tax5 ?tax6 ?tax7 ?tax8 ?tax9 ?tax10 ?tax11 ?tax12 ?tax13 ?tax14 ?tax15
from <http://ddbj.nig.ac.jp/ontologies/taxonomy>
where {
?taxlevel1 a ddbjtax:Taxon ;
rdfs:subClassOf  taxon:33090 .
?taxlevel2 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel1 .
  ?taxlevel3 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel2 .
  ?taxlevel4 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel3 .
  ?taxlevel5 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel4 .
    ?taxlevel6 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel5 .
    ?taxlevel7 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel6 .
    ?taxlevel8 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel7 .
    ?taxlevel9 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel8 .
    ?taxlevel10 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel9 .
      ?taxlevel11 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel10 .
      ?taxlevel12 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel11 .
      ?taxlevel13 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel12 .
      ?taxlevel14 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel13 .
      ?taxlevel15 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel14 .
  
  BIND(IRI(REPLACE(STR(?taxlevel1), "http://identifiers.org/taxonomy/","")) AS ?tax1)
  BIND(IRI(REPLACE(STR(?taxlevel2), "http://identifiers.org/taxonomy/","")) AS ?tax2)
    BIND(IRI(REPLACE(STR(?taxlevel3), "http://identifiers.org/taxonomy/","")) AS ?tax3)
    BIND(IRI(REPLACE(STR(?taxlevel4), "http://identifiers.org/taxonomy/","")) AS ?tax4)
    BIND(IRI(REPLACE(STR(?taxlevel5), "http://identifiers.org/taxonomy/","")) AS ?tax5)
      BIND(IRI(REPLACE(STR(?taxlevel6), "http://identifiers.org/taxonomy/","")) AS ?tax6)
      BIND(IRI(REPLACE(STR(?taxlevel7), "http://identifiers.org/taxonomy/","")) AS ?tax7) 
      BIND(IRI(REPLACE(STR(?taxlevel8), "http://identifiers.org/taxonomy/","")) AS ?tax8)
      BIND(IRI(REPLACE(STR(?taxlevel9), "http://identifiers.org/taxonomy/","")) AS ?tax9)
      BIND(IRI(REPLACE(STR(?taxlevel10), "http://identifiers.org/taxonomy/","")) AS ?tax10)
        BIND(IRI(REPLACE(STR(?taxlevel11), "http://identifiers.org/taxonomy/","")) AS ?tax11)
        BIND(IRI(REPLACE(STR(?taxlevel12), "http://identifiers.org/taxonomy/","")) AS ?tax12)
        BIND(IRI(REPLACE(STR(?taxlevel13), "http://identifiers.org/taxonomy/","")) AS ?tax13)
        BIND(IRI(REPLACE(STR(?taxlevel14), "http://identifiers.org/taxonomy/","")) AS ?tax14)
        BIND(IRI(REPLACE(STR(?taxlevel15), "http://identifiers.org/taxonomy/","")) AS ?tax15)
  
}

```

## `list`
```javascript
({taxonids}) => {
  
 let tree = [
  ];
  
 taxonids.results.bindings.map(d =>
    tree.push(
   d.tax1.value,
   d.tax2.value,
   d.tax3.value,
   d.tax4.value,
   d.tax5.value,
   d.tax6.value,
   d.tax7.value,
   d.tax8.value,
   d.tax9.value,
   d.tax10.value,
   d.tax11.value,
   d.tax12.value,
   d.tax13.value,
   d.tax14.value,
   d.tax15.value,
  )
      );

  return tree;
}

```

##`removeDuplicate`
```javascript
({list}) => {

  const tree2 = Array.from(new Set(list))

   return tree2 ;
}
  
```

