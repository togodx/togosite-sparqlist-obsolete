# togoKey ID list without filtering (hgnc, uniprot, pdb, mondo) (未対応 pubchem_compound)

## Parameters

* `togoKey`
  * example: hgnc, uniprot, pdb, mondo, (pubchem_compound)

## `key`
```javascript
({togoKey}) => {
  let obj = {};
  obj[togoKey] = true;
  return obj;
}
```


## Endpoint
https://integbio.jp/togosite/sparql

## `idList`
```sparql
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX dcterm: <http://purl.org/dc/terms/>
PREFIX oboowl: <http://www.geneontology.org/formats/oboInOwl#>
SELECT DISTINCT ?id
{{#if key.hgnc}}
FROM <http://rdf.integbio.jp/dataset/togosite/hgnc>
WHERE {
  ?uri a obo:SO_0000704 .
  BIND(REPLACE(STR(?uri), "http://identifiers.org/hgnc/", "") AS ?id)
}
{{/if}}
{{#if key.uniprot}}
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
  ?uri a uniprot:Protein ;
         uniprot:organism taxon:9606 ;
         uniprot:proteome ?proteome .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
  BIND(REPLACE(STR(?uri), "http://purl.uniprot.org/uniprot/", "") AS ?id)
}
{{/if}}
{{#if key.pdb}}
FROM <http://rdf.integbio.jp/dataset/togosite/pdbj>
WHERE {
  ?uri a pdbo:datablock ;
       dcterm:identifier ?id .
   FILTER(REGEX(STR(?uri), "https://rdf.wwpdb.org/pdb/"))
}
{{/if}} 
{{#if key.mondo}}
FROM <http://rdf.integbio.jp/dataset/togosite/mondo>
WHERE {
  ?uri oboowl:id ?id_a .
  FILTER (REGEX (STR (?uri), "MONDO_"))
  BIND(REPLACE(?id_a, "MONDO:", "") AS ?id)
}
{{/if}}   
```

## `return`
```javascript
({idList})=>{
  return idList.results.bindings.map(d => d.id.value);
}
```