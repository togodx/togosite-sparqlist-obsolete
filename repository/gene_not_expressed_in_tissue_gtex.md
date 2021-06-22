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

## `gene_list` queryIdsを配列に

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

## `category_list` categoryIds を配列に

```javascript
({categoryIds}) => {
  categoryIds = categoryIds.replace(/,/g, " ")
  if (categoryIds.match(/[^\s]/)) return categoryIds.split(/\s+/);
  return false;
}
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

SELECT DISTINCT (COUNT(DISTINCT ?ensg) AS ?count) ?tissue_name
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refex_gtex_v8_summary> {
    ?refex a refexo:RefExEntry ;
           sio:SIO_000216 ?exp_bn ;
           refexo:isMeasurementOf ?ensg ;
           refexo:refexSample ?refexs .
    ?exp_bn a refexo:logTPMMax ;
            sio:SIO_000300 0 .
  }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refexsample_gtex_v8_summary> {
    ?refexs dcterms:description ?tissue_name ;
            schema:additionalProperty ?sample_bn .
    ?sample_bn schema:name "tissue" ;
               schema:valueReference ?tissue_uri .
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
        categoryId: d.description.value,
        label: d.description_label.value,
        count: Number(d.count.value)
      };
    });
  }
};
```