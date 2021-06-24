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
PREFIX ensg: <http://rdf.ebi.ac.uk/resource/ensembl/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>

{{#if mode}}
SELECT DISTINCT ?tissue ?tissue_id ?label ?ensg
{{else}}
SELECT ?tissue ?tissue_id ?label (COUNT(DISTINCT ?ensg) AS ?count) (SAMPLE(?child_tissue) AS ?child_example)
{{/if}}
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refex_gtex_v8_summary> {
    {{#if input_genes}}
    VALUES ?ensg { {{#each input_genes}} ensg:{{this}} {{/each}} }
    {{/if}}
    ?refex a refexo:RefExEntry ;
           sio:SIO_000216 ?exp_bn ;
           refexo:isMeasurementOf ?ensg ;
           refexo:refexSample ?refexs .
    ?exp_bn a refexo:logTPMMax ;
            sio:SIO_000300 0 .
            #sio:SIO_000300 ?logtpmmax .
    #FILTER(?logtpmmax > 0 && ?logtpmmax < 0.1)
  }

  {{#if input_tissues}}
  VALUES {{#if mode}} ?tissue {{else}} ?parent {{/if}} { {{#each input_tissues_uberon}} obo:{{this}} {{/each}} {{#each input_tissues_efo}} efo:{{this}} {{/each}} }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refexo_tissue_classifications> {
  {{#unless mode}}
    ?tissue skos:broader ?parent .
  {{/unless}}
    ?narrower_tissue skos:broader* ?tissue .
  }
  {{else}}
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refexo_tissue_classifications> {
    ?narrower_tissue skos:broader* ?tissue .
    FILTER NOT EXISTS {
      ?tissue skos:broader ?parent .
    }
  }
  {{/if}}
  OPTIONAL {
    GRAPH <http://rdf.integbio.jp/dataset/togosite/refexo_tissue_classifications> {
      ?child_tissue skos:broader ?tissue .
    }
  }

  GRAPH <http://rdf.integbio.jp/dataset/togosite/refexsample_gtex_v8_summary> {
    ?refexs schema:additionalProperty ?sample_bn .
    VALUES ?sample_type { "tissue" "cell type"}
    ?sample_bn schema:name ?sample_type ;
               schema:valueReference ?narrower_tissue .
  }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/ensembl> {
    ?ensg a ?type .
    VALUES ?type { enso:protein_coding enso:lncRNA enso:processed_pseudogene enso:unprocessed_pseudogene
                   enso:transcribed_unprocessed_pseudogene enso:LRG_gene enso:transcribed_processed_pseudogene enso:IG_V_pseudogene
                   enso:IG_V_gene enso:TR_V_gene enso:transcribed_unitary_pseudogene enso:unitary_pseudogene enso:TR_J_gene
                   enso:polymorphic_pseudogene enso:IG_D_gene enso:TR_V_pseudogene enso:pseudogene enso:IG_J_gene enso:IG_C_gene
                   enso:IG_C_pseudogene enso:TR_C_gene enso:IG_J_pseudogene enso:TR_D_gene enso:TR_J_pseudogene
                   enso:translated_processed_pseudogene enso:IG_pseudogene enso:translated_unprocessed_pseudogene }
  }

  {
    GRAPH <http://rdf.integbio.jp/dataset/togosite/efo> {
      ?tissue rdfs:label ?label .
    }
  }
  UNION
  {
    GRAPH <http://rdf.integbio.jp/dataset/togosite/uberon> {
      ?tissue rdfs:label ?label .
    }
  }

  BIND(REPLACE(REPLACE(STR(?tissue), "http://purl.obolibrary.org/obo/", ""), "http://www.ebi.ac.uk/efo/", "") AS ?tissue_id)
}

```

## `return`

```javascript
({ main, mode }) => {
  if (mode === "idList") {
    return Array.from(new Set(
      main.results.bindings.map((elem) =>
        elem.ensg.value.replace("http://rdf.ebi.ac.uk/resource/ensembl/", "")
      )
    ));
  } else if (mode === "objectList") {
    return main.results.bindings.map((elem) => ({
      id: elem.ensg.value.replace("http://rdf.ebi.ac.uk/resource/ensembl/", ""),
      attribute: {
        categoryId: elem.tissue.value
          .replace("http://purl.obolibrary.org/obo/", "")
          .replace("http://www.ebi.ac.uk/efo/", ""),
        uri: elem.tissue.value,
        label: capitalize(modifyLabel(elem.label.value))
      }
    }));
  } else {
    return main.results.bindings.map((elem) => ({
      categoryId: elem.tissue.value
        .replace("http://purl.obolibrary.org/obo/", "")
        .replace("http://www.ebi.ac.uk/efo/", ""),
      label: capitalize(modifyLabel(elem.label.value)),
      count: Number(elem.count.value),
      hasChild: Boolean(elem.child_example)
    })).sort((a, b) => a.label.toLowerCase() < b.label.toLowerCase() ? -1 : 1);
  }

  function modifyLabel(label) {
    const map = new Map([
      ["breast epithelium", "breast"],
      ["right lobe of liver", "liver"],
      ["upper lobe of left lung", "lung"],
      ["anterior lingual gland", "minor salivary gland"],
      ["gastrocnemius medialis", "skeletal muscle"],
      ["body of pancreas", "pancreas"],
      ["Peyer's patch", "small intestine"],
      ["venous blood", "blood"],
      ["skin of body", "skin"],
    ]);

    if (map.has(label)) {
      return map.get(label);
    } else {
      return label;
    }
  }

  function capitalize(str) {
    return str.charAt(0).toUpperCase() + str.substring(1);
  }
};
```