# test togoid

* source か target のどちらかは ensembl_gene

## Parameters
* `source`
  * example: ensembl_gene
* `target`
  * example: ensembl_transcript
* `ids`
  * example: ENSG00000006634,ENSG00000023909,ENSG00000040633,ENSG00000060339,ENSG00000060566,ENSG00000066044,ENSG00000067141,ENSG00000072736,ENSG00000073150,ENSG00000075886

## `idArray`
```javascript
({ids}) => {
  ids = ids.replace(/,/g," ")
  if (ids.match(/[^\s]/) && ids != "undefined") return ids.split(/\s+/);
  return false;
}
```

## `type`
```javascript
({source, target})=>{
  let obj = {};
  if (source == "ensembl_gene") obj.source = true;
  return obj;
}
```

## Endpoint
https://integbio.jp/rdf/ebi/sparql
## `link`
```sparql
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX ensembl_gene: <http://rdf.ebi.ac.uk/resource/ensembl/>
PREFIX ensembl_transcript: <http://rdf.ebi.ac.uk/resource/ensembl.transcript/>
SELECT DISTINCT ?id1 ?id2
WHERE {
  {{#if type.source}}
  VALUES ?id1_uri { {{#each idArray}} ensembl_gene:{{this}} {{/each}} }
  {{else}}
  VALUES ?id2_uri { {{#each idArray}} ensembl_transcript:{{this}} {{/each}} }  
  {{/if}}
  ?id1_uri obo:RO_0002162 taxon:9606 ;
     dc:identifier ?id1 .
  ?id2_uri obo:SO_transcribed_from ?id1_uri ;
        dc:identifier ?id2 .
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
