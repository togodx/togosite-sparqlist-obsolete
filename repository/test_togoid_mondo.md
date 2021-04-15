# test togoid

* source か target のどちらかは mondo

## Parameters
* `source`
  * example: mondo
* `target`
  * example: ncbigene
* `ids`
  * example: MONDO_0011102,MONDO_0012865,MONDO_0008648

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
  if (source == "mondo") obj.source = true;
  return obj;
}
```

## Endpoint
https://integbio.jp/togosite/sparql
## `link`
```sparql
PREFIX mo: <http://med2rdf/ontology/medgen#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX mondo: <http://purl.obolibrary.org/obo/>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>
SELECT DISTINCT ?id1 ?id2
FROM <http://rdf.integbio.jp/dataset/togosite/medgen>
WHERE {
  {{#if type.source}}
  VALUES ?id1_uri { {{#each idArray}} {{this}} {{/each}} }
  {{else}}
  VALUES ?id2_uri { {{#each idArray}} {{this}} {{/each}} }  
  {{/if}}
  ?medgen mo:mgconso [
    dct:source mo:MONDO ;
    rdfs:seeAlso ?id1_uri 
  ] .
  ?id2_uri obo:RO_0003302 ?medgen .
  BIND (STRAFTER(STR(?id2_uri), "ncbigene/") AS ?id2)
  BIND (STRAFTER(STR(?id1_uri), "obo/") AS ?id1)
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
