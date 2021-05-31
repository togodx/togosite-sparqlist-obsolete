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
PREFIX up: <http://purl.uniprot.org/core/>
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
    ?uniprot_uri up:recommendedName/up:fullName ?uniprot_name .
  }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/chebi> {
    ?chebi_uri rdfs:label ?chebi_name .
  }
}
```

## `ppi`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
SELECT DISTINCT ?type ?target ?target_name
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
  {{#if type_chk.uniprot}}
  VALUES ?type { up:Non_Self_Interaction up:Self_Interaction }
  VALUES ?uniprot { uniprot:{{id}} }
  ?uniprot a up:Protein ;
           up:interaction [ a ?type ;
                            ^up:interaction ?target 
                          ] .
  ?target up:recommendedName/up:fullName ?target_name .
  {{/if}}
}
```

## `assay`
```sparql
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX up: <http://purl.uniprot.org/core/>
SELECT DISTINCT ?uniprot ?uniprot_name ?chembl ?chembl_label ?assay_type
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
  {{#if type_chk.uniprot}}
  VALUES ?uniprot { uniprot:{{id}} }
  ?chembl a cco:SmallMolecule ;
          skos:prefLabel ?chembl_label ;
          cco:hasActivity/cco:hasAssay ?assay .
  ?assay cco:hasTarget/skos:exactMatch/skos:exactMatch ?uniprot .
  OPTIONAL { ?assay cco:assayType ?assay_type . }
  ?uniprot a cco:UniprotRef .
  GRAPH <http://rdf.integbio.jp/dataset/togosite/uniprot> {
    ?uniprot up:recommendedName/up:fullName ?uniprot_name .
  }
  {{/if}}
}
```
