# Genes expressed in tissues (GTEx ver6)（小野・池田・千葉）(mode対応版)
This query allows **multiple tissue flags** for each gene.

## Endpoint

https://orth.dbcls.jp/sparql-dev

## Parameters
* `categoryIds` (type: UBERON or EFO)
  * example: UBERON_0010414,UBERON_0002369,UBERON_0001496,EFO_0000572
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
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX refexo: <http://purl.jp/bio/01/refexo#>
PREFIX ensembl: <http://identifiers.org/ensembl/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX efo: <http://www.ebi.ac.uk/efo/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

{{#if mode}}
SELECT DISTINCT ?ensg ?tissue ?label
{{else}}
SELECT ?tissue ?label ?has_child (COUNT(DISTINCT ?ensg) AS ?count)
{{/if}}
WHERE {
  {
    {{#if input_genes}}
    VALUES ?ensg { {{#each input_genes}} ensembl:{{this}} {{/each}} }
    {{/if}}
    {{#if input_tissues}}
    VALUES ?tissue { {{#each input_tissues_uberon}} obo:{{this}} {{/each}} {{#each input_tissues_efo}} efo:{{this}} {{/each}} }
    {{/if}}
    GRAPH <https://refex.dbcls.jp/rdf/tissue_specific_genes_gtex_v6> {
      ?ensg refexo:isPositivelySpecificTo ?tissue .
    }
    FILTER NOT EXISTS {
      ?tissue skos:broader ?parent .
    }
    BIND(0 AS ?has_child)
    ?tissue rdfs:label ?label .
  }
  UNION
  {
    {{#if input_genes}}
    VALUES ?ensg { {{#each input_genes}} ensembl:{{this}} {{/each}} }
    {{/if}}
    {{#if input_tissues}} #親がいて子がいない
    VALUES ?parent { {{#each input_tissues_uberon}} obo:{{this}} {{/each}} {{#each input_tissues_efo}} efo:{{this}} {{/each}} }
    GRAPH <https://refex.dbcls.jp/rdf/tissue_specific_genes_gtex_v6> {
      ?ensg refexo:isPositivelySpecificTo ?tissue .
    }
    ?tissue skos:broader ?parent .
    BIND(0 AS ?has_child)
    {{else}} #子がいて親がいない
    GRAPH <https://refex.dbcls.jp/rdf/tissue_specific_genes_gtex_v6> {
      ?ensg refexo:isPositivelySpecificTo ?child .
    }
    ?child skos:broader ?tissue .
    BIND(1 AS ?has_child)
    {{/if}}
    ?tissue rdfs:label ?label .
  }
}ORDER BY ?label


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
          .replace("http://purl.obolibrary.org/obo/", "")
          .replace("http://www.ebi.ac.uk/efo/", ""),
        uri: elem.tissue.value,
        label: makeLabel(elem.label.value)
      }
    }));
  } else {
    return main.results.bindings.map((elem) => ({
      categoryId: elem.tissue.value
        .replace("http://purl.obolibrary.org/obo/", "")
        .replace("http://www.ebi.ac.uk/efo/", ""),
      label: makeLabel(elem.label.value),
      count: elem.count.value,
      hasChild: Boolean(Number(elem.has_child.value))
    }));
  }
};

function makeLabel(label) {
  if (label === "breast epithelium") {
    return "breast";
  } else if (label === "right lobe of liver") {
    return "liver";
  } else if (label === "upper lobe of left lung") {
    return "lung";
  } else if (label === "anterior lingual gland") {
    return "minor salivary gland";
  } else if (label === "gastrocnemius medialis") {
    return "skeletal muscle";
  } else if (label === "body of pancreas") {
    return "pancreas";
  } else if (label === "Peyer's patch") {
    return "small intestine";
  } else if (label === "venous blood") {
    return "blood";
  } else if (label === "skin of body") {
    return "skin";
  } else {
    return label;
  }
};
```