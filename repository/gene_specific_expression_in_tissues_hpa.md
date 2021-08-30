# Genes specifically expressed in tissues (HPA)（小野・池田・千葉）
This query allows **multiple tissue flags** for each gene.

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters
* `categoryIds` (type: CALOHA)
  * example: TS-0892,TS-0013,TS-0440
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

## `main`

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX ensembl: <http://identifiers.org/ensembl/>
PREFIX refexo: <http://purl.jp/bio/01/refexo#>
PREFIX obo: <http://purl.obolibrary.org/obo/caloha.obo#>
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
    GRAPH <http://rdf.integbio.jp/dataset/togosite/hpa_tissue_specificity> {
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
          .replace("http://purl.obolibrary.org/obo/caloha.obo#", ""),
        uri: elem.tissue.value,
        label: elem.label.value
      }
    }));
  } else {
    return main.results.bindings.map((elem) => ({
      categoryId: elem.tissue.value
        .replace("http://purl.obolibrary.org/obo/caloha.obo#", ""),
      label: elem.label.value,
      count: Number(elem.count.value)
    })).sort((a, b) => a.label.toLowerCase() < b.label.toLowerCase() ? -1 : 1);
  }
};
```