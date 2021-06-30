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
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX efo: <http://www.ebi.ac.uk/efo/>
PREFIX enso: <http://rdf.ebi.ac.uk/terms/ensembl/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX refexo:  <http://purl.jp/bio/01/refexo#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX schema: <http://schema.org/>
PREFIX ensg: <http://rdf.ebi.ac.uk/resource/ensembl/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>

{{#if mode}}
SELECT DISTINCT ?tissue ?ensg
{{else}}
SELECT ?tissue (COUNT (DISTINCT ?ensg) AS ?count)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/refexsample_gtex_v8_summary>
FROM <http://rdf.integbio.jp/dataset/togosite/refex_gtex_v8_summary>
FROM <http://rdf.integbio.jp/dataset/togosite/ensembl>
FROM <http://rdf.integbio.jp/dataset/togosite/refexo_tissue_classifications>
WHERE {
  {
    SELECT DISTINCT ?refex ?ensg ?tissue
    WHERE { 
      {
        SELECT DISTINCT ?refex ?ensg ?tissue
        WHERE {
          {
            SELECT DISTINCT ?narrower_tissue ?tissue
            WHERE {
              {
                SELECT DISTINCT ?narrower_tissue
                WHERE {
                  [] refexo:refexSample/schema:additionalProperty/schema:valueReference ?narrower_tissue .
                }
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
                MINUS { ?tissue skos:broader ?parent . }
              }
              {{/if}}
              #OPTIONAL { ?child_tissue skos:broader ?tissue . }
              #?tissue rdfs:label ?label .
              #BIND(REPLACE(REPLACE(STR(?tissue), "http://purl.obolibrary.org/obo/", ""), "http://www.ebi.ac.uk/efo/", "") AS ?tissue_id)
            }
          }
          {{#if input_genes}}
          VALUES ?ensg { {{#each input_genes}} ensg:{{this}} {{/each}} }
          {{/if}}
          ?refex refexo:isMeasurementOf ?ensg ;
                 refexo:refexSample/schema:additionalProperty [
                   schema:valueReference ?narrower_tissue ;
                   schema:name ?sample_type
                 ] .
          VALUES ?sample_type { "tissue" "cell type"}
        }
      }
      GRAPH <http://rdf.integbio.jp/dataset/togosite/refex_gtex_v8_summary> {
        ?refex sio:SIO_000216 [
          a refexo:logTPMMax ;
          sio:SIO_000300 ?logtpmmax 
        ] . 
      }
      FILTER(?logtpmmax = 0)
    }
  }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/ensembl> {
    ?ensg a ?type .
    VALUES ?type { enso:lncRNA obo:SO_0001217 obo:SO_0000336 enso:TEC }
    MINUS { ?ensg a enso:rRNA_pseudogene }
  }
}
```

## `tissue_label`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX refexo:  <http://purl.jp/bio/01/refexo#>
PREFIX schema: <http://schema.org/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>

SELECT DISTINCT ?tissue ?label  (SAMPLE(?child_tissue) AS ?child_example)
FROM <http://rdf.integbio.jp/dataset/togosite/refexsample_gtex_v8_summary>
FROM <http://rdf.integbio.jp/dataset/togosite/refex_gtex_v8_summary>
FROM <http://rdf.integbio.jp/dataset/togosite/refexo_tissue_classifications>
FROM <http://rdf.integbio.jp/dataset/togosite/efo> 
FROM <http://rdf.integbio.jp/dataset/togosite/uberon>
WHERE {
  {
    SELECT DISTINCT ?narrower_tissue
    WHERE {
      [] refexo:refexSample/schema:additionalProperty/schema:valueReference ?narrower_tissue .
    }
  }
  ?narrower_tissue skos:broader* ?tissue .
  OPTIONAL { ?child_tissue skos:broader ?tissue . }
  ?tissue rdfs:label ?label .
}
```

## `return`

```javascript
({ main, tissue_label, mode }) => {
  let urlToLabel = new Map(tissue_label.results.bindings.map(o => [o.tissue.value, o.label.value]))
  let urlToChild = new Map(tissue_label.results.bindings.map(o => [o.tissue.value, o.child_example?.value]))
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
        label: capitalize(modifyLabel(urlToLabel.get(elem.tissue.value)))
      }
    }));
  } else {
    return main.results.bindings.map((elem) => ({
      categoryId: elem.tissue.value
        .replace("http://purl.obolibrary.org/obo/", "")
        .replace("http://www.ebi.ac.uk/efo/", ""),
      label: capitalize(modifyLabel(urlToLabel.get(elem.tissue.value))),
      count: Number(elem.count.value),
      hasChild: Boolean(urlToChild.get(elem.tissue.value))
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
