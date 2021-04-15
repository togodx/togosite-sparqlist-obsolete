# タンパク質(PDBまたはUniprot)のエントリ内容を返す（八塚）

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters
* `id`
  * default: P06493
  * example: P06493, 6TIW
* `type`
  * default: uniprot
  * example: uniprot, pdb
  
## `idDict` returns an id type object

```javascript
({id, type}) => {
  var obj = {};
  switch (type) {
    case 'uniprot':
      obj.uniprot = id;
      break;
    case 'pdb':
      obj.pdb = id;
      break;
  }
  return obj;
}
```

## `main`

```sparql
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX core: <http://purl.uniprot.org/core/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX pdbr: <https://rdf.wwpdb.org/pdb/>
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>

{{#if idDict.uniprot }}	#uniprot->pdb
SELECT DISTINCT ?uniprot_id ?uniprot_full_name ?uniprot_short_name ?uniprot_mass Count(?uniprot_citation) AS ?uniprot_citation_number ?pdb_id ?pdb_title ?pdb_methods ?pdb_polypeptide_number ?pdb_pH_str ?pdb_keywords ?pdb_text ?pdb_spieces ?pdb_host
{{/if}}
{{#if idDict.pdb }}	#pdb->uniprot
SELECT DISTINCT ?pdb_id ?uniprot_full_name ?uniprot_short_name ?uniprot_mass Count(?uniprot_citation) AS ?uniprot_citation_number ?uniprot_id ?pdb_title ?pdb_methods ?pdb_polypeptide_number ?pdb_pH_str ?pdb_keywords ?pdb_text ?pdb_spieces ?pdb_host
{{/if}}

WHERE{
  {{#if idDict.uniprot }}	#uniprot->pdb
  VALUES ?uniprot { uniprot:{{idDict.uniprot}} }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/togoid> {
    ?uniprot rdfs:seeAlso ?pdb .
    FILTER(STRSTARTS(STR(?pdb), STR(pdbr:)))
  }
  {{/if}}
  {{#if idDict.pdb }}	#pdb->uniprot
  VALUES ?pdb { pdbr:{{idDict.pdb}} }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/togoid> {
    ?pdb rdfs:seeAlso ?uniprot .
    FILTER(STRSTARTS(STR(?uniprot), STR(uniprot:)))
  }
  {{/if}}
  
  #uniprotエントリ情報
  GRAPH <http://rdf.integbio.jp/dataset/togosite/uniprot> {  
    ?uniprot a core:Protein .
    ?uniprot core:recommendedName ?rname .
    ?rname core:fullName ?uniprot_full_name .
    OPTIONAL {?rname core:shortName ?sname .}
    BIND(CONCAT("", ?sname) AS ?short_name)
    BIND((IF(STRLEN(?short_name)=0,"-", ?short_name)) AS ?uniprot_short_name)
    ?uniprot core:sequence/core:mass ?uniprot_mass .
    OPTIONAL {?uniprot core:citation ?uniprot_citation .}
    BIND(REPLACE(STR(?uniprot), uniprot:, "") AS ?uniprot_id)
  }
    
  #pdbエントリ情報
  GRAPH <http://rdf.integbio.jp/dataset/togosite/pdbj> {
     ?pdb a pdbo:datablock;
     dc:title ?pdb_title;
     pdbo:has_exptlCategory/pdbo:has_exptl/pdbo:exptl.method ?pdb_methods .  

      OPTIONAL {?pdb pdbo:has_exptl_crystal_growCategory/pdbo:has_exptl_crystal_grow/pdbo:exptl_crystal_grow.pH ?pH .}
      OPTIONAL {?pdb pdbo:has_em_bufferCategory/pdbo:has_em_buffer/pdbo:em_buffer.pH ?pH .}
      BIND(CONCAT("", ?pH) AS ?pH_str)
      BIND((IF(STRLEN(STR(?pH_str))=0,"N/A", ?pH_str)) AS ?pdb_pH_str)


     ?pdb pdbo:has_struct_keywordsCategory/pdbo:has_struct_keywords/pdbo:struct_keywords.pdbx_keywords ?pdb_keywords .
     ?pdb pdbo:has_struct_keywordsCategory/pdbo:has_struct_keywords/pdbo:struct_keywords.text ?pdb_text.

     ?pdb pdbo:has_entity_src_genCategory/pdbo:has_entity_src_gen/pdbo:entity_src_gen.pdbx_gene_src_scientific_name ?pdb_spieces.
     ?pdb pdbo:has_entity_src_genCategory/pdbo:has_entity_src_gen/pdbo:entity_src_gen.pdbx_host_org_scientific_name ?pdb_host.
    
     {Select DISTINCT ?pdb COUNT(?polypeptide)AS ?pdb_polypeptide_number {
     ?pdb pdbo:has_entity_polyCategory/pdbo:has_entity_poly ?polypeptide.
     ?polypeptide rdfs:seeAlso ?uniprot_link . #DNAのentryを排除
     }}
     
     BIND(REPLACE(STR(?pdb), pdbr:, "") AS ?pdb_id)
  }
}
```

## `return`
```javascript
({ main }) => {
  return main.results.bindings.map((elm) => ({
    uniprot_id: elm.uniprot_id.value,
    uniprot_full_name: elm.uniprot_full_name.value,
    uniprot_short_name: elm.uniprot_short_name.value,
    uniprot_mass: elm.uniprot_mass.value,
    uniprot_citation_number: elm.uniprot_citation_number.value,
    pdb_id: elm.pdb_id.value,
    pdb_title: elm.pdb_title.value,
    pdb_methods: elm.pdb_methods.value,
    pdb_polypeptide_number: elm.pdb_polypeptide_number.value,
    pdb_pH_str: elm.pdb_pH_str.value,
    pdb_keywords: elm.pdb_keywords.value,
    pdb_text: elm.pdb_text.value,
    pdb_spieces: elm.pdb_spieces.value,
    pdb_host: elm.pdb_host.value
  }));
}

```