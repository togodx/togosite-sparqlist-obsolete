# Gene attributes from PDB RDF（井手)

## Endpoint

https://integbio.jp/rdf/sparql

## Parameters
* `id`
  * default: 6TIW
  * example: 6TIW

## `main`

```sparql
PREFIX pdbr: <https://rdf.wwpdb.org/pdb/>
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>

SELECT DISTINCT ?id ?title ?methods ?polypeptide_number ?pH_str ?keywords ?text ?species ?host
                (GROUP_CONCAT(DISTINCT ?uniprot_id; separator=", ") AS ?uniprot_ids)
                (GROUP_CONCAT(DISTINCT ?uniprot_desc; separator=", ") AS ?uniprot_descs)
WHERE {
  VALUES ?PDBentry { pdbr:{{id}} }
  ?PDBentry dc:title ?title ;
            pdbo:has_exptlCategory/pdbo:has_exptl/pdbo:exptl.method ?methods .  
  OPTIONAL { ?PDBentry pdbo:has_exptl_crystal_growCategory/pdbo:has_exptl_crystal_grow/pdbo:exptl_crystal_grow.pH ?pH . }
  OPTIONAL { ?PDBentry pdbo:has_em_bufferCategory/pdbo:has_em_buffer/pdbo:em_buffer.pH ?pH . }
  BIND(IF(STRLEN(?pH)>0, STR(?pH),"N/A") AS ?pH_str)

  ?PDBentry pdbo:has_struct_keywordsCategory/pdbo:has_struct_keywords/pdbo:struct_keywords.pdbx_keywords ?keywords .
  ?PDBentry pdbo:has_struct_keywordsCategory/pdbo:has_struct_keywords/pdbo:struct_keywords.text ?text.
  ?PDBentry pdbo:has_entity_src_genCategory/pdbo:has_entity_src_gen/pdbo:entity_src_gen.pdbx_gene_src_scientific_name ?species.
  ?PDBentry pdbo:has_entity_src_genCategory/pdbo:has_entity_src_gen/pdbo:entity_src_gen.pdbx_host_org_scientific_name ?host.
    
  {
    SELECT DISTINCT ?PDBentry COUNT(?polypeptide) AS ?polypeptide_number {
     ?PDBentry pdbo:has_entity_polyCategory/pdbo:has_entity_poly ?polypeptide.
     ?polypeptide rdfs:seeAlso ?uniprot_link . #DNAのentryを排除
    }
  }
  OPTIONAL {
    ?PDBentry pdbo:has_entityCategory/pdbo:has_entity ?entity_each .
    ?entity_each pdbo:referenced_by_struct_ref/pdbo:link_to_uniprot ?uniprot_id .
    ?entity_each pdbo:entity.pdbx_description ?uniprot_desc .
  }
  BIND(REPLACE(STR(?PDBentry), pdbr:, "") AS ?id)
}
```

## `return`

```javascript
({ main }) => {
  return main.results.bindings.map((elem) => ({
    pdb_id: elem.id.value,
    title: elem.title.value,
    methods: elem.methods.value,
    polypeptide_number : elem.polypeptide_number.value,
    pH_str : elem.pH_str.value,
    keywords : elem.keywords.value,
    text: elem.text.value,
    species: elem.species.value,
    host: elem.host.value,
    uniprot_id: elem.uniprot_ids.value,
    uniprot_desc: elem.uniprot_descs.value
  }));
};
```