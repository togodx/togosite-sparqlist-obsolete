# Genes specifically expressed in tissues (HPA)（小野・池田・千葉）(mode対応版)
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
PREFIX hpa: <http://www.proteinatlas.org/>
PREFIX : <http://www.proteinatlas.org/about/nanopubs/>
PREFIX np: <http://www.nanopub.org/nschema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX nif: <http://ontology.neuinfo.org/NIF/Backend/NIF-Quality.owl#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX wp: <http://vocabularies.wikipathways.org/wp#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?tissue (COUNT(DISTINCT ?gene) as ?count)
FROM <http://rdf.integbio.jp/dataset/togosite/protein-atlas>
FROM <http://rdf.integbio.jp/dataset/togosite/caloha>
WHERE {
  ?gene a wp:GeneProduct .
  GRAPH ?g {
    ?gene obo:BFO_0000066 ?tissue ; # Occurs in
          nif:nlx_qual_1010003 "Medium"^^xsd:string . #?exp . # Expression level
  }
}ORDER BY ?count
```

## `tissueIdArray`
```javascript
({main}) => {
  return main.results.bindings.map(d => d.tissue.value.replace("http://www.proteinatlas.org/", ""));
}
```

## `tissueLabel`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX tissue: <http://purl.obolibrary.org/obo/caloha.obo#>

SELECT DISTINCT ?tissue ?label
FROM <http://rdf.integbio.jp/dataset/togosite/caloha>
WHERE {
  VALUES ?tissue { {{#each tissueIdArray}} tissue:{{this}} {{/each}} }
  ?tissue rdfs:label ?label .
} ORDER BY ?label
```

## `labelMap`
```javascript
({tissueLabel}) => {
  var labelMap = tissueLabel.result.bindings.map((elem) => ([
    elem.tissue.value, elem.label.value
  ]));
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