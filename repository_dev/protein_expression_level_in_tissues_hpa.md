# Protein expression level in tissues (HPA)（小野・池田・千葉）

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters
* `categoryIds` (type: CALOHA)
  * example: TS-0892,TS-0013,TS-0440
* `queryIds` (type: ensembl_gene)
  * example: ENSG00000000005,ENSG00000002587,ENSG00000003989
* `mode`
  * example: idList, objectList
* `level` (required)
  * example: Not_detected, Low, Medium, High
  * default: High

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

{{#if mode}}
SELECT DISTINCT ?tissue ?gene
{{else}}
SELECT DISTINCT ?tissue (COUNT(DISTINCT ?gene) as ?count)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/protein-atlas>
FROM <http://rdf.integbio.jp/dataset/togosite/caloha>
WHERE {
  {{#if input_genes}}
  VALUES ?gene { {{#each input_genes}} hpa:{{this}} {{/each}} }
  {{/if}}
  {{#if input_tissues}}
  VALUES ?tissue { {{#each input_tissues}} hpa:{{this}} {{/each}}  }
  {{/if}}
  ?gene a wp:GeneProduct .
  GRAPH ?g {
    ?gene obo:BFO_0000066 ?tissue ; # Occurs in
          nif:nlx_qual_1010003 "{{level}}"^^xsd:string . #?exp . # Expression level
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
ラベル取得も一つの SPARQL で行おうとするとなぜか遅いので分割した
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

## `return`

```javascript
({ main, mode, tissueLabel }) => {
  const labelMap = new Map(tissueLabel.results.bindings.map((elem) => ([ elem.tissue.value, elem.label.value ])));
  if (mode === "idList") {
    return Array.from(new Set(
      main.results.bindings.map((elem) =>
        elem.gene.value.replace("http://www.proteinatlas.org/", "")
      )
    ));
  } else if (mode === "objectList") {
    return main.results.bindings.map((elem) => ({
      id: elem.gene.value.replace("http://www.proteinatlas.org/", ""),
      attribute: {
        categoryId: elem.tissue.value
          .replace("http://www.proteinatlas.org/", ""),
        uri: elem.tissue.value,
        label: labelMap.get(elem.tissue.value
                            .replace("http://www.proteinatlas.org/", "http://purl.obolibrary.org/obo/caloha.obo#"))
      }
    }));
  } else {
    return main.results.bindings.map((elem) => ({
      categoryId: elem.tissue.value
        .replace("http://www.proteinatlas.org/", ""),
      label: labelMap.get(elem.tissue.value
                          .replace("http://www.proteinatlas.org/", "http://purl.obolibrary.org/obo/caloha.obo#")),
      count: Number(elem.count.value)
    })).sort((a, b) => a.label.toLowerCase() < b.label.toLowerCase() ? -1 : 1);
    //})).sort((a, b) => a.count < b.count ? -1 : 1);
  }
};
```