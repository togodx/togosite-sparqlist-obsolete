# togokey label (aggregate SPARQList) ラベル取得

- ラベル無し key は "key" セクションでフラグ

## Parameters

* `togoKey` hgnc, ncbigene, ensembl_gene, ensembl_transcript, uniprot, pdb, pubchem_compound, chembl_compound, chebi, mondo, mesh, hp, nando
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
  if (togoKey == "glytoucan" || togoKey == "togovar") obj.unlabeledKey = true;
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
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX hop: <http://purl.org/net/orthordf/hOP/ontology#>
PREFIX hgnc: <http://identifiers.org/hgnc/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>
PREFIX ensembl_gene: <http://rdf.ebi.ac.uk/resource/ensembl/>
PREFIX ensembl_transcript: <http://rdf.ebi.ac.uk/resource/ensembl.transcript/>
PREFIX pdb: <https://rdf.wwpdb.org/pdb/>
PREFIX chembl_compound: <http://rdf.ebi.ac.uk/resource/chembl/molecule/>
PREFIX chebi: <http://purl.obolibrary.org/obo/CHEBI_>
PREFIX pubchem_compound: <http://rdf.ncbi.nlm.nih.gov/pubchem/compound/CID>
PREFIX mondo: <http://purl.obolibrary.org/obo/MONDO_>
PREFIX mesh: <http://id.nlm.nih.gov/mesh/>
PREFIX hp: <http://purl.obolibrary.org/obo/HP_>
PREFIX nando: <http://nanbyodata.jp/ontology/nando#>
SELECT ?id ?label
{{#if key.hgnc}}
FROM <http://rdf.integbio.jp/dataset/togosite/hgnc>
{{/if}}
{{#if key.ncbigene}}
FROM <http://rdf.integbio.jp/dataset/togosite/homo_sapiens_gene_info>
{{/if}}
{{#if key.ensembl_gene}}
FROM <http://rdf.integbio.jp/dataset/togosite/ensembl>
{{/if}}
{{#if key.ensembl_transcript}}
FROM <http://rdf.integbio.jp/dataset/togosite/ensembl>
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
FROM <http://rdf.integbio.jp/dataset/togosite/pubchem>
{{/if}}
{{#if key.mondo}}
FROM <http://rdf.integbio.jp/dataset/togosite/mondo>
{{/if}}
{{#if key.mesh}}
FROM <http://rdf.integbio.jp/dataset/togosite/mesh>
{{/if}}
{{#if key.hp}}
FROM <http://rdf.integbio.jp/dataset/togosite/hpo>
{{/if}}
{{#if key.nanod}}
FROM <http://rdf.integbio.jp/dataset/togosite/nando>
{{/if}}
WHERE {
 {{#unless key.unlabeledKey}}
  VALUES ?uri { {{#each queryArray}} {{../togoKey}}:{{this}} {{/each}} }
  {{#if key.hgnc}}
  ?uri rdfs:label ?label .
  {{/if}}
  {{#if key.ncbigene}}
  ?uri hop:fullName ?label .
  {{/if}}
  {{#if key.ensembl_gene}}
  ?uri rdfs:label ?label .
  {{/if}}
  {{#if key.ensembl_transcript}}
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
  ?uri sio:has-attribute [
    a sio:CHEMINF_000382 ;
    sio:has-value ?label 
  ] .
  {{/if}}
  {{#if key.mondo}}
  ?uri rdfs:label ?label .
  {{/if}}
  {{#if key.mesh}}
  ?uri rdfs:label ?label .
  {{/if}}
  {{#if key.hp}}
  ?uri rdfs:label ?label .
  {{/if}}
  {{#if key.nando}}
  ?uri rdfs:label ?label .
  FILTER(LANG(?label) = "en")
  {{/if}}
  BIND(REPLACE(STR(?uri), {{togoKey}}:, "") AS ?id)
 {{/unless}}
}
```

## `return`
```javascript
({label})=>{
  let obj = {};
  for (let d of label.results.bindings) {
    if (!d.id) break;
    obj[d.id.value] = d.label.value;
  }
  return obj;
}
```