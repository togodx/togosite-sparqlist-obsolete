# togokey label (aggregate SPARQList) ラベル取得

## Parameters

* `togoKey` hgnc, uniprot, pdb, pubchem_compound, mondo, (chembl_comppound, chebi)
  * default: hgnc
* `queryIds`
  * default: ["4942","5344","6148", "6265","6344","6677","6735","10593","10718","10876"]

## `queryArray`
```javascript
({queryIds})=>{
  return JSON.parse(queryIds);
}
```

## `key`
```javascript
({togoKey})=>{
  let obj = {};
  obj[togoKey] = true;
  return obj;
}
```

## Endpoint
https://integbio.jp/togosite/sparql

## `label`
```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX core: <http://purl.uniprot.org/core/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX hgnc: <http://identifiers.org/hgnc/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX pdb: <https://rdf.wwpdb.org/pdb/>
PREFIX chembl_compound: <http://rdf.ebi.ac.uk/resource/chembl/molecule/>
PREFIX chebi: <http://purl.obolibrary.org/obo/CHEBI_>
PREFIX pubchem_compound: <http://rdf.ncbi.nlm.nih.gov/pubchem/compound/CID>
PREFIX mondo: <http://purl.obolibrary.org/obo/MONDO_>
SELECT ?id ?label
{{#if key.hgnc}}
FROM <http://rdf.integbio.jp/dataset/togosite/hgnc>
{{/if}}
{{#if key.uniprot}}
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
{{/if}}
{{#if key.pdb}}
FROM <http://rdf.integbio.jp/dataset/togosite/pdbj>
{{/if}}
{{#if key.chembl_compound}}
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>
{{/if}}
{{#if key.chebi}}
FROM <http://rdf.integbio.jp/dataset/togosite/chebi>
{{/if}}
{{#if key.pubchem_compound}}
FROM <http://rdf.integbio.jp/dataset/togosite/chebi>
FROM <http://rdf.integbio.jp/dataset/togosite/pubchem>
{{/if}}
{{#if key.mondo}}
FROM <http://rdf.integbio.jp/dataset/togosite/mondo>
{{/if}}
WHERE {
  VALUES ?uri { {{#each queryArray}} {{../togoKey}}:{{this}} {{/each}} }
  {{#if key.hgnc}}
  ?uri rdfs:label ?label .
  {{/if}}
  {{#if key.uniprot}}
  ?uri core:mnemonic ?label .
  {{/if}}
  {{#if key.pdb}}
  ?uri dc:title ?label .
  {{/if}}
  {{#if key.chembl_compound}}
  ?uri skos:prefLabel ?label .
  {{/if}}
  {{#if key.chebi}}
  ?uri rdfs:label ?label .
  {{/if}}
  {{#if key.pubchem_compound}}
  ?uri rdf:type/rdfs:label ?label .
  {{/if}}
  {{#if key.mondo}}
  ?uri rdfs:label ?label .
  {{/if}}
  BIND(REPLACE(STR(?uri), {{togoKey}}:, "") AS ?id)
}
```

## `return`
```javascript
({label})=>{
  let obj = {};
  for (let d of label.results.bindings) {
    obj[d.id.value] = d.label.value;
  }
  return obj;
}
```