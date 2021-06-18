# Genes expressed in tissues (GTEx ver6)（小野・池田・千葉）(mode対応版)
This query allows **multiple tissue flags** for each gene.

Forked from `gtex6_tissues`. Modified to include "low specificity" as a result.

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
({ queryIds }) => {
  queryIds = queryIds.replace(/,/g, " ");
  if (queryIds.match(/\S/)) {
    return queryIds.split(/\s+/);
  }
};
```

## `input_tissues`
```javascript
({ categoryIds }) => {
  categoryIds = categoryIds.replace(/,/g, " ");
  categoryIds = categoryIds.replace("low_specificity", "");
  if (categoryIds.match(/\S/)) {
    return categoryIds.split(/\s+/);
  }
};
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

## `input_tissues_low_spec`
```javascript
({ categoryIds }) => {
  categoryIds = categoryIds.replace(/,/g, " ");
  if (categoryIds.split(/\s+/).includes("low_specificity")) {
    return ["low_specificity"];
  }
};
```

## `flag_do_main`
```javascript
({ input_tissues, input_tissues_low_spec }) => {
  if (!input_tissues && input_tissues_low_spec) {
    return false;
  } else {
    return true;
  }
};
```

## `flag_do_low_spec`
```javascript
({ input_tissues, input_tissues_low_spec }) => {
  if (input_tissues && !input_tissues_low_spec) {
    return false;
  } else {
    return true;
  }
};
```

## `main`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX ensembl: <http://identifiers.org/ensembl/>
PREFIX refexo: <http://purl.jp/bio/01/refexo#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX efo: <http://www.ebi.ac.uk/efo/>

{{#if mode}}
SELECT DISTINCT ?tissue ?tissue_id ?label ?ensg
{{else}}
SELECT ?tissue ?tissue_id ?label (COUNT(DISTINCT ?ensg) AS ?count) (SAMPLE(?child_tissue) AS ?child_example)
{{/if}}
WHERE {
  {{#if flag_do_main}}
  {{#if input_genes}}
  VALUES ?ensg { {{#each input_genes}} ensembl:{{this}} {{/each}} }
  {{/if}}
  {{#if input_tissues}}
  VALUES {{#if mode}} ?tissue {{else}} ?parent {{/if}} { {{#each input_tissues_uberon}} obo:{{this}} {{/each}} {{#each input_tissues_efo}} efo:{{this}} {{/each}} }
  {{#unless mode}} ?tissue skos:broader ?parent . {{/unless}}
  {{else}}
  FILTER NOT EXISTS {
    ?tissue skos:broader ?parent .
  }
  {{/if}}
  ?narrower_tissue skos:broader* ?tissue .
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refex_tissue_specific_genes_gtex_v6> {
    ?ensg refexo:isPositivelySpecificTo ?narrower_tissue .
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
  OPTIONAL {
    ?child_tissue skos:broader ?tissue .
  }
  BIND(REPLACE(REPLACE(STR(?tissue), "http://purl.obolibrary.org/obo/", ""), "http://www.ebi.ac.uk/efo/", "") AS ?tissue_id)
  {{/if}}
}
```

## `low_spec`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX ensembl: <http://identifiers.org/ensembl/>
PREFIX refexo: <http://purl.jp/bio/01/refexo#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX efo: <http://www.ebi.ac.uk/efo/>

{{#if mode}}
SELECT DISTINCT ?tissue ?tissue_id ?label ?ensg
{{else}}
SELECT ?tissue ?tissue_id ?label (COUNT(DISTINCT ?ensg) AS ?count) ?child_example
{{/if}}
WHERE {
  {{#if flag_do_low_spec}}
  BIND("" AS ?tissue)
  BIND("low_specificity" AS ?tissue_id)
  BIND("Low specificity" AS ?label)
  {{#if input_genes}}
  VALUES ?ensg { {{#each input_genes}} ensembl:{{this}} {{/each}} }
  {{/if}}
  ?ensg a refexo:GTEx_v6_ts_evaluated_gene .
  GRAPH <http://rdf.integbio.jp/dataset/togosite/refex_tissue_specific_genes_gtex_v6> {
    FILTER NOT EXISTS {
      ?ensg refexo:isPositivelySpecificTo ?t .
    }
  }
  {{/if}}
}
```

## `return`

```javascript
({ main, mode, low_spec, flag_do_main, flag_do_low_spec }) => {
  var results = []
  if (flag_do_main) {
    results = main.results.bindings.sort((a, b) => a.label.value.toLowerCase() < b.label.value.toLowerCase() ? -1 : 1);
  }
  if (flag_do_low_spec) {
    results = results.concat(low_spec.results.bindings).filter((x)=>Object.keys(x).length!=0);
  }
  if (mode === "idList") {
    return Array.from(new Set(
      results.map((elem) =>
        elem.ensg.value.replace("http://identifiers.org/ensembl/", "")
      )
    ));
  } else if (mode === "objectList") {
    return results.map((elem) => ({
      id: elem.ensg.value.replace("http://identifiers.org/ensembl/", ""),
      attribute: {
        categoryId: elem.tissue_id.value,
        uri: elem.tissue.value,
        label: capitalize(modifyLabel(elem.label.value))
      }
    }));
  } else {
    return results.map((elem) => ({
      categoryId: elem.tissue_id.value,
      label: capitalize(modifyLabel(elem.label.value)),
      count: Number(elem.count.value),
      hasChild: Boolean(elem.child_example)
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
