# Tissues where a gene is not expressed （池田）

## Description

- Data sources
    - [GTEx version 8](https://gtexportal.org/home/datasets)
- Query
    -  Input
        - Ensembl Gene ID
    - Output
        - RefEx Sample ID for tissues

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters

* `categoryIds` (type: RefEx Sample ID)
  * example: RES00003651,RES00003662,RES00003690,RES00003701
* `queryIds` (type: ensembl_gene)
  * example: ENSG00000230748,ENSG00000213070,ENSG00000271527,ENSG00000227776,ENSG00000230898
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
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX refexo:  <http://purl.jp/bio/01/refexo#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX schema: <http://schema.org/>
PREFIX ensg: <http://rdf.ebi.ac.uk/resource/ensembl/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>

{{#if mode}}
SELECT DISTINCT ?sample_label ?sample_id ?ensg
{{else}}
SELECT ?sample_label ?sample_id (COUNT (DISTINCT ?ensg) AS ?count)
{{/if}}
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refex_gtex_v8_summary> {
    {{#if input_genes}}
    VALUES ?ensg { {{#each input_genes}} ensg:{{this}} {{/each}} }
    {{/if}}
    ?refex a refexo:RefExEntry ;
           sio:SIO_000216 [
             a refexo:logTPMMax ;
             sio:SIO_000300 0
             #sio:SIO_000300 ?logtpmmax
           ] ;
           refexo:isMeasurementOf ?ensg ;
           refexo:refexSample ?refexs .
    #FILTER(?logtpmmax >= 0 && ?logtpmmax < 1)
  }
  
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refexsample_gtex_v8_summary> {
    {{#if input_tissues}}
    VALUES ?sample_id { {{#each input_tissues}} "{{this}}" {{/each}}  }
    {{/if}}
    ?refexs dcterms:description ?sample_label ;
            dcterms:identifier ?sample_id .
  }

  GRAPH <http://rdf.integbio.jp/dataset/togosite/ensembl> {
    ?ensg a ?type .
    VALUES ?type { enso:lncRNA obo:SO_0001217 obo:SO_0000336 enso:TEC }
    MINUS { ?ensg a enso:rRNA_pseudogene }
  }

}ORDER BY ?sample_label
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
        categoryId: elem.sample_id.value,
        uri: "http://refex.dbcls.jp/sample/" + elem.sample_id.value,
        label: elem.sample_label.value
      }
    }));
  } else {
    return main.results.bindings.map((elem) => ({
      categoryId: elem.sample_id.value,
      label: elem.sample_label.value,
      count: Number(elem.count.value),
      hasChild: false
    }));
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
