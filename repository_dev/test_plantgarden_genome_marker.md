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
select distinct ?tax23
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
        ?taxlevel16 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel15 .
        ?taxlevel17 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel16 .
        ?taxlevel18 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel17 .
        ?taxlevel19 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel18 .
       ?taxlevel20 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel19 .
         ?taxlevel21 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel20 .
         ?taxlevel22 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel21 .
         ?taxlevel23 a ddbjtax:Taxon ;
rdfs:subClassOf  ?taxlevel22 .
  
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
          BIND(IRI(REPLACE(STR(?taxlevel16), "http://identifiers.org/taxonomy/","")) AS ?tax16)
          BIND(IRI(REPLACE(STR(?taxlevel17), "http://identifiers.org/taxonomy/","")) AS ?tax17)
          BIND(IRI(REPLACE(STR(?taxlevel18), "http://identifiers.org/taxonomy/","")) AS ?tax18)
          BIND(IRI(REPLACE(STR(?taxlevel19), "http://identifiers.org/taxonomy/","")) AS ?tax19)
          BIND(IRI(REPLACE(STR(?taxlevel20), "http://identifiers.org/taxonomy/","")) AS ?tax20)
          BIND(IRI(REPLACE(STR(?taxlevel21), "http://identifiers.org/taxonomy/","")) AS ?tax21) 
          BIND(IRI(REPLACE(STR(?taxlevel22), "http://identifiers.org/taxonomy/","")) AS ?tax22)
          BIND(IRI(REPLACE(STR(?taxlevel23), "http://identifiers.org/taxonomy/","")) AS ?tax23)  
}

```

## `list`
```javascript
({taxonids}) => {
  
 let tree = [
  ];
  
 taxonids.results.bindings.map(d =>
    tree.push(
 
      d.tax23.value,
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

