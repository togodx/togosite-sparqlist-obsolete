# test togoid

* source か target のどちらかは uniprot

## Parameters
* `source`
  * example: uniprot
* `target`
  * example: hgnc, ncbigene, ensembl_gene, ensembl_transcript, pdb, chembl_target
* `ids`
  * example: Q9NYF8,Q4V339,A6NCE7,A7E2F4,P69849,A6NN73,Q92928,Q5T1J5,P0C7P4,Q6DN03,P09874,Q08211,Q5T4S7,P12270,Q9UPN3,P07814,P53621,P49321,P0C629,Q9BZK8,Q9BY65

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
  if (source == "hgnc" || target == "hgnc") obj.hgnc = true;
  if (source == "ncbigene" || target == "ncbigene") obj.ncbigene = true;
  if (source == "ensembl_gene" || target == "ensembl_gene") obj.ensembl_gene = true;
  if (source == "ensembl_transcript" || target == "ensembl_transcript") obj.ensembl_transcript = true;
  if (source == "pdb" || target == "pdb") obj.pdb = true;
  if (source == "chembl_target" || target == "chembl_target") obj.chembl_target = true;
  
  if (source == "uniprot") obj.source = true;
  return obj;
}
```

## Endpoint
https://integbio.jp/togosite/sparql
## `link`
```sparql
PREFIX core: <http://purl.uniprot.org/core/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX hgnc: <http://purl.uniprot.org/hgnc/>
PREFIX ncbigene: <http://purl.uniprot.org/geneid/>
PREFIX ensembl_gene: <http://rdf.ebi.ac.uk/resource/ensembl/>
PREFIX ensembl_transcript: <http://rdf.ebi.ac.uk/resource/ensembl.transcript/>
PREFIX pdb: <http://rdf.wwpdb.org/pdb/>
PREFIX chembl_target: <http://rdf.ebi.ac.uk/resource/chembl/target/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX db: <http://purl.uniprot.org/database/>
SELECT DISTINCT (REPLACE(STR(?id1_uri), uniprot:, "") AS ?id1) (REPLACE(STR(?id2_uri), {{#if type.source}}{{target}}{{else}}{{source}}{{/if}}:, "") AS ?id2)
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
  {{#if type.source}}
  VALUES ?id1_uri { {{#each idArray}} {{this}} {{/each}} }
  {{else}}
  VALUES ?id2_uri { {{#each idArray}} {{this}} {{/each}} }  
  {{/if}}
  ?id1_uri a core:Protein ;
           core:organism taxon:9606 .
  {{#if type.hgnc}}
  ?id1_uri rdfs:seeAlso ?id2_uri .
  ?id2_uri core:database db:HGNC .
  {{/if}}
  {{#if type.ncbigene}}
  ?id1_uri rdfs:seeAlso ?id2_uri .
  ?id2_uri core:database db:GeneID .
  {{/if}}
  {{#if type.ensembl_gene}}
  ?id1_uri rdfs:seeAlso [ 
    core:database db:Ensembl ;
    core:transcribedFrom ?id2_uri
  ] .
  {{/if}}
  {{#if type.ensembl_transcript}}
  ?id1_uri rdfs:seeAlso ?id2_uri .
  ?id2_uri core:database db:Ensembl .
  {{/if}}
  {{#if type.pdb}}
  ?id1_uri rdfs:seeAlso ?id2_uri .
  ?id2_uri core:database db:PDB .
  {{/if}}
  {{#if type.chembl_target}}
  ?id1_uri rdfs:seeAlso ?id2_uri .
  ?id2_uri core:database db:ChEMBL .
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
