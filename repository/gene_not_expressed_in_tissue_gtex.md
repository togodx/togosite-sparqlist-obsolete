# Tissues where a gene is not expressed （池田）

## Description

- Data sources
    - [GTEx version 8](https://gtexportal.org/home/datasets)
    - Definition of gene biotypes is described [here](http://useast.ensembl.org/info/genome/genebuild/biotypes.html).
- Query
    -  Input
        - Ensembl Gene ID
    - Output
        - Gene type

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters

* `categoryIds` (type: UBERON or EFO)
  * example: UBERON_0010414,UBERON_0002369,UBERON_0001496,EFO_0000572
* `queryIds` (type: ensembl_gene)
  * example: ENSG00000000005,ENSG00000002587,ENSG00000003989,ENSG00000049883
* `mode`
  * example: idList, objectList

## `input_genes`

```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/\s/g, "");
  if (queryIds) {
    return queryIds.split(",");
  } else {
    return false;
  }
};
```

## `input_tissues`

```javascript
({categoryIds}) => {
  categoryIds = categoryIds.replace(/,/g, " ")
  if (categoryIds.match(/[^\s]/)) return categoryIds.split(/\s+/);
  return false;
}
```

## `input_tissues_uberon`
```javascript
({ input_tissues }) => {
  if (input_tissues) {
    return input_tissues.filter((x) => x.match(/^UBERON/));
  }
};
```

## `input_tissues_efo`
```javascript
({ input_tissues }) => {
  if (input_tissues) {
    return input_tissues.filter((x) => x.match(/^EFO/));
  }
};
```

## `main`

```sparql
# Endpoint: https://integbio.jp/togosite/sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX enso: <http://rdf.ebi.ac.uk/terms/ensembl/>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX taxonomy: <http://identifiers.org/taxonomy/>
PREFIX refexo:  <http://purl.jp/bio/01/refexo#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX schema: <http://schema.org/>
PREFIX ensg: <http://rdf.ebi.ac.uk/terms/ensembl/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>

{{#if mode}}
SELECT DISTINCT ?tissue ?tissue_id ?tissue_name ?ensg
{{else}}
SELECT ?tissue ?tissue_id ?tissue_name (COUNT(DISTINCT ?ensg) AS ?count) (SAMPLE(?child_tissue) AS ?child_example)
{{/if}}
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refex_gtex_v8_summary> {
    {{#if input_genes}}
    VALUES ?ensg { {{#each input_genes}} ensembl:{{this}} {{/each}} }
    {{/if}}
    ?refex a refexo:RefExEntry ;
           sio:SIO_000216 ?exp_bn ;
           refexo:isMeasurementOf ?ensg ;
           refexo:refexSample ?refexs .
    ?exp_bn a refexo:logTPMMax ;
            sio:SIO_000300 0 .
  }

  {{#if input_tissues}}
  VALUES {{#if mode}} ?tissue {{else}} ?parent {{/if}} { {{#each input_tissues_uberon}} obo:{{this}} {{/each}} {{#each input_tissues_efo}} efo:{{this}} {{/each}} }
  {{#unless mode}}
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refexo_tissue_classifications> {
    ?tissue skos:broader ?parent .
    ?narrower_tissue skos:broader* ?tissue .
    OPTIONAL {
      ?child_tissue skos:broader ?tissue .
    }
  }
  {{/unless}}
  {{else}}
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refexo_tissue_classifications> {
    FILTER NOT EXISTS {
      ?tissue skos:broader ?parent .
    }
    ?narrower_tissue skos:broader* ?tissue .
    OPTIONAL {
      ?child_tissue skos:broader ?tissue .
    }
  }
  {{/if}}

  GRAPH <http://rdf.integbio.jp/dataset/togosite/refexsample_gtex_v8_summary> {
    ?refexs dcterms:description ?tissue_name ;
            schema:additionalProperty ?sample_bn .
    ?sample_bn schema:name "tissue" ;
               schema:valueReference ?narrower_tissue .
  }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/ensembl> {
    ?ensg a ?type ;
          rdfs:label ?label .
    VALUES ?type { enso:protein_coding enso:lncRNA enso:processed_pseudogene enso:unprocessed_pseudogene
                   enso:transcribed_unprocessed_pseudogene enso:LRG_gene enso:transcribed_processed_pseudogene enso:IG_V_pseudogene
                   enso:IG_V_gene enso:TR_V_gene enso:transcribed_unitary_pseudogene enso:unitary_pseudogene enso:TR_J_gene
                   enso:polymorphic_pseudogene enso:IG_D_gene enso:TR_V_pseudogene enso:pseudogene enso:IG_J_gene enso:IG_C_gene
                   enso:IG_C_pseudogene enso:TR_C_gene enso:IG_J_pseudogene enso:TR_D_gene enso:TR_J_pseudogene
                   enso:translated_processed_pseudogene enso:IG_pseudogene enso:translated_unprocessed_pseudogene }
    FILTER(CONTAINS(STR(?type), "http://rdf.ebi.ac.uk/terms/ensembl/"))
  }
  BIND(REPLACE(REPLACE(STR(?tissue), "http://purl.obolibrary.org/obo/", ""), "http://www.ebi.ac.uk/efo/", "") AS ?tissue_id)
}

```

## `return`

```javascript
({mode, main}) => {
  if (mode == "objectList") {
    return main.results.bindings.map(d => {
      return {
        id: d.id.value,
        attribute: {
          categoryId: d.description.value,
          uri: d.type.value,
          label: d.description_label.value
        }
      };
    });
  }
  if (mode == "idList") {
    return Array.from(new Set(main.results.bindings.map(d => d.id.value))); // unique
  }
  else {
    return main.results.bindings.map(d => {
      return {
        categoryId: d.tissue_id.value,
        label: d.tissue_name.value,
        count: Number(d.count.value)
      };
    });
  }
};
```