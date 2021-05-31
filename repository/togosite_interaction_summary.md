# Interaction attribute（守屋）

## Parameters
* `id`
  * default: P12270
  * example: P12270, 15996
* `type`
  * default: uniprot
  * example: uniprot chebi
  
## `type_chk`
```javascript
({ type }) => {
  let obj = {};
  obj[type] = true;
  return obj;
};
```
## Endpoint
https://integbio.jp/togosite/sparql

## `reaction`
```sparql
PREFIX biopax: <http://www.biopax.org/release/biopax-level3.owl#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX core: <http://purl.uniprot.org/core/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT ?reaction ?label ?uniprot ?uniprot_name ?uniprot_uri ?chebi ?chebi_name ?chebi_uri
FROM <http://rdf.integbio.jp/dataset/togosite/reactome>
FROM <http://rdf.integbio.jp/dataset/togosite/chebi>
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
  VALUES ?reaction_type { biopax:BiochemicalReaction biopax:TemplateReaction biopax:Degradation }
  VALUES ?has_component { biopax:left biopax:right biopax:product }
  {{#if type_chk.uniprot}}
  VALUES ?uniprot { "{{id}}"^^xsd:string }
  {{/if}}
  {{#if type_chk.chebi}}
    VALUES ?chebi { "{{id}}"^^xsd:string }
  {{/if}}
  ?react a ?reaction_type ;
         biopax:displayName ?label ;
         biopax:xref [
           biopax:db "Reactome"^^xsd:string ;
           biopax:id ?reaction ;
         ] ;
         ?has_component ?complex .
  ?complex biopax:component*/biopax:entityReference/biopax:xref [
    biopax:db "UniProt"^^xsd:string ;
    biopax:id ?uniprot
  ] .
  ?complex biopax:component*/biopax:entityReference/biopax:xref [
    biopax:db "ChEBI"^^xsd:string ;
    biopax:id ?chebi
  ] .
  BIND(IRI(CONCAT ("http://purl.uniprot.org/uniprot/", ?uniprot)) AS ?uniprot_uri)
  BIND(IRI(CONCAT ("http://purl.obolibrary.org/obo/", REPLACE(?chebi, ":", "_"))) AS ?chebi_uri)
  GRAPH <http://rdf.integbio.jp/dataset/togosite/uniprot> {
    ?uniprot_uri core:recommendedName/core:fullName ?uniprot_name .
  }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/chebi> {
    ?chebi_uri rdfs:label ?chebi_name .
  }
}
```
