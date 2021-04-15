# test togoid

* source か target のどちらかは hgnc

## Parameters
* `source`
  * example: hgnc
* `target`
  * example: ncbigene, ensembl_gene, uniprot
* `ids`
  * example: 10050,10065,10322,1034,10378,10379,10380,10436,10477,10603,10606,10810,10827,1084,10840,10852,1087,11179,11317,11359,11389,11448,11449,11450,11713,11741,1176,118,11831

## `idArray`
```javascript
({ids, source}) => {
  ids = ids.replace(/,/g," ")
  if (ids.match(/[^\s]/) && ids != "undefined") return ids.split(/\s+/).map(d=>source + ":" + d);
  return false;
}
```

## `type`
```javascript
({source, target})=>{
  let obj = {};
  if (source == "ncbigene" || target == "ncbigene") obj.ncbigene = true;
  if (source == "ensembl_gene" || target == "ensembl_gene") obj.ensembl_gene = true;
  if (source == "uniprot" || target == "uniprot") obj.uniprot = true;
  
  if (source == "hgnc") obj.source = true;
  return obj;
}
```

## Endpoint
http://sparql.med2rdf.org/sparql
## `link`
```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX m2r: <http://med2rdf.org/ontology/med2rdf#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX idt: <http://identifiers.org/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX hgnc: <http://identifiers.org/hgnc/>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>
PREFIX ensembl_gene: <http://identifiers.org/ensembl/>
SELECT DISTINCT ?id1 ?id2
FROM <http://med2rdf.org/graph/hgnc>
WHERE {
  {{#if type.source}}
  VALUES ?id1_uri { {{#each idArray}} {{this}} {{/each}} }
  {{else}}
  VALUES ?id2_uri { {{#each idArray}} {{this}} {{/each}} }  
  {{/if}}
  ?id1_uri a obo:SO_0000704, m2r:Gene ;
    dct:identifier ?id1 ;
    rdfs:seeAlso ?id2_uri .
  ?id2_uri dct:identifier ?id2 ;
  {{#if type.ncbigene}}
            a idt:ncbigene .
  {{/if}}
  {{#if type.ensembl_gene}}
            a idt:ensembl .
  {{/if}}
  {{#if type.uniprot}}
            a idt:uniprot .
  {{/if}}
}
```

## `return`
```javascript
({type, link})=>{
  if (type.source) return link.results.bindings.map(d => {
    return {
      source: d.id1.value,
      target: d.id2.value
    }
  });
  return link.results.bindings.map(d => {
    return {
      source: d.id2.value,
      target: d.id1.value
    }
  });
}
```
