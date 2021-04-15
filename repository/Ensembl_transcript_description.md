# Ensembl transcript description（井手）

## Endpoint

https://integbio.jp/rdf/ebi/sparql

## Parameters

* `categoryId` -(type:種別/機能)
  * example: protein_coding,lncRNA,retained_intron,processed_transcript,nonsense_mediated_decay,processed_pseudogene,unprocessed_pseudogene,misc_RNA,snRNA,miRNA,TEC,transcribed_unprocessed_pseudogene,snoRNA,transcribed_processed_pseudogene,rRNA_pseudogene,IG_V_pseudogene,IG_V_gene,TR_V_gene,transcribed_unitary_pseudogene,polymorphic_pseudogene,unitary_pseudogene,non_stop_decay,TR_J_gene,rRNA,IG_D_gene,pseudogene,scaRNA,TR_V_pseudogene,IG_C_gene,IG_J_gene,Mt_tRNA,IG_C_pseudogene,TR_C_gene,ribozyme,IG_J_pseudogene,sRNA,TR_D_gene,TR_J_pseudogene,translated_processed_pseudogene,Mt_rRNA,IG_pseudogene,vault_RNA,scRNA,translated_unprocessed_pseudogene
* `queryIds` -(type: Ensembl)
  * example: ENST00000460472
* `mode` 
  * example: idList, objectList

## `gene_list`
```javascript
({ queryIds }) => {
  queryIds = queryIds.replace(/\s/g, "");
  if (queryIds) {
    return queryIds.split(",");
  } else {
    return false;
  }
};
```

## `category_list`
- categoryId を配列に
```javascript
({categoryId}) => {
  categoryId = categoryId.replace(/,/g," ")
  if (categoryId.match(/[^\s]/)) return categoryId.split(/\s+/);
  return false;
}
```

## `main`

```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX identifiers: <http://identifiers.org/>
PREFIX biohack: <http://biohackathon.org/resource/faldo#>
PREFIX enst: <http://rdf.ebi.ac.uk/resource/ensembl.transcript/>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX biohack: <http://biohackathon.org/resource/faldo#>


{{#if mode}}
  SELECT DISTINCT ?description ?des ?id ?gene_name ?gene_symbol ?description_label
{{else}}
  SELECT DISTINCT ?description count(?des) AS ?count ?description_label
{{/if}}
  WHERE {
   {{#if gene_list}}
   VALUES ?entry { {{#each gene_list}} enst:{{this}} {{/each}} }
   {{/if}}
  ?entry a ?des .
  ?entry obo:RO_0002162 taxon:9606 .
  filter contains(STR(?entry), "http://rdf.ebi.ac.uk/resource/ensembl.transcript/ENST")
  filter contains(STR(?des), "terms/ensembl/")
     {{#if mode}}   
     ?entry rdfs:label ?gene_symbol .
     ?entry obo:SO_transcribed_from/dc:description ?gene_name .
     {{/if}}  
  BIND(REPLACE(STR(?entry), enst:, "") AS ?id)
  BIND(REPLACE(STR(?des), "http://rdf.ebi.ac.uk/terms/ensembl/", "") AS ?description)
  BIND(REPLACE((?description), "_", " ") AS ?description_label)
   {{#if category_list}}
    VALUES ?description_query { {{#each category_list}} "{{this}}" {{/each}} } 
    filter (?description IN (?description_query))
   {{/if}}
}
Order by DESC(?count)
                   
```

## `return`

```javascript
({mode, main }) => {
  if (mode == "objectList") return main.results.bindings.map(d=>{ 
    return {
      id: d.id.value, 
        attribute: {
             categoryId: d.description.value,
             uri: d.des.value, 
             label: d.description_label.value
                   }
                };
              });
  if (mode == "idList") return Array.from(new Set(main.results.bindings.map(d=>d.id.value.replace("https://rdf.wwpdb.org/pdb/", "")))); // unique 
  return main.results.bindings.map(d=>{ 
     return {
       categoryId: d.description.value, 
       label: d.description_label.value, 
       count: Number(d.count.value)
       };
   });	
};
```