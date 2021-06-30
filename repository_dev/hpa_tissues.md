# Genes expressed in tissues (HPA)（小野・池田・千葉）(mode対応版)
This query allows **multiple tissue flags** for each gene.

## Endpoint

https://orth.dbcls.jp/sparql-dev

## Parameters
* `categoryIds` (type: UBERON)
  * example: UBERON_0001013,UBERON_0002369,UBERON_0000178
* `queryIds` (type: ensembl_gene)
  * example: ENSG00000000005,ENSG00000002587,ENSG00000003989
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

## `main`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX ensembl: <http://identifiers.org/ensembl/>
PREFIX refexo: <http://purl.jp/bio/01/refexo#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX efo: <http://www.ebi.ac.uk/efo/>

{{#if mode}}
SELECT DISTINCT ?tissue ?label ?ensg
{{else}}
SELECT ?tissue ?label (COUNT(DISTINCT ?ensg) AS ?count)
{{/if}}
WHERE {
  {{#if input_genes}}
  VALUES ?ensg { {{#each input_genes}} ensembl:{{this}} {{/each}} }
  {{/if}}
  {{#if input_tissues}}
  VALUES ?tissue { {{#each input_tissues}} obo:{{this}} {{/each}} }
  {{/if}}
  GRAPH <https://refex.dbcls.jp/rdf/hpa_tissue_specificity> {
    ?ensg refexo:isPositivelySpecificTo ?tissue .
  }
  ?tissue rdfs:label ?label .
}
```

## `return`

```javascript
({ main, mode }) => {
  if (mode === "idList") {
    return Array.from(new Set(
      main.results.bindings.map((elem) =>
        elem.ensg.value.replace("http://identifiers.org/ensembl/", "")
      )
    ));
  } else if (mode === "objectList") {
    return main.results.bindings.map((elem) => ({
      id: elem.ensg.value.replace("http://identifiers.org/ensembl/", ""),
      attribute: {
        categoryId: elem.tissue.value
          .replace("http://purl.obolibrary.org/obo/", ""),
        uri: elem.tissue.value,
        label: modifyLabel(elem.label.value)
      }
    }));
  } else {
    return main.results.bindings.map((elem) => ({
      categoryId: elem.tissue.value
        .replace("http://purl.obolibrary.org/obo/", ""),
      label: modifyLabel(elem.label.value),
      count: Number(elem.count.value)
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
};
```