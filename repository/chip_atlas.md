# Genes with hypothetical upstream TFs in ChIP-Atlas（大田・池田・小野・千葉）(mode対応版)

## Description

- Data sources
    - ChIP-Atlas: [http://dbarchive.biosciencedbc.jp/kyushu-u/hg38/target/](http://dbarchive.biosciencedbc.jp/kyushu-u/hg38/target/)
    - The genes in `<Transcription Factor>.10.tsv` were defined to be "Genes with hypothetical upstream TF".

- Query
    - The output ID 1 and 2 are assigned to "Genes with hypothetical upstream TF" and Genes without hypothetical upstream TF" respectively.
    - Input
        - Ensembl gene ID
    - Output
        - with or without hypothetical upstream TF

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters
* `categoryIds` (type: 1 or 2)
  * example: 1
* `queryIds` (type: ensembl gene)
  * example: ENSG00000000005,ENSG00000002587,ENSG00000115942
* `mode` (type: string)
  * example: idList, objectList

## `input_genes`
```javascript
({ queryIds }) => {
  queryIds = queryIds.replace(/,/g, " ");
  if (queryIds.match(/[^\s]/)) {
    return queryIds.split(/\s+/);
  } else {
    return false;
  }
};
```

## `input_categories`
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
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX refexo: <http://purl.jp/bio/01/refexo#>
PREFIX ensembl: <http://identifiers.org/ensembl/>

{{#if mode}}
SELECT DISTINCT ?id ?ensg
{{else}}
SELECT ?id (COUNT(DISTINCT ?ensg) AS ?count)
{{/if}}
WHERE {
  {
    {{#if input_genes}}
    VALUES ?ensg { {{#each input_genes}} ensembl:{{this}} {{/each}} }
    {{/if}}
    ?upstream obo:RO_0002428 ?ensg .
    BIND("1" AS ?id)
  }
  UNION
  {
    {{#if input_genes}}
    VALUES ?ensg { {{#each input_genes}} ensembl:{{this}} {{/each}} }
    {{/if}}
    ?ensg refexo:ncbigene ?ncbigene .
    FILTER NOT EXISTS {
      ?upstream obo:RO_0002428 ?ensg .
    }
    BIND("2" AS ?id)
  }
  {{#if input_categories}}
  VALUES ?id { {{#each input_categories}} "{{this}}" {{/each}} }
  {{/if}}
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
        categoryId: elem.id.value,
        label: makeLabel(elem.id.value)
      }
    }));
  } else {
    return main.results.bindings.map((elem) => ({
      categoryId: elem.id.value,
      label: makeLabel(elem.id.value),
      count: Number(elem.count.value),
    }));
  }

  function makeLabel(id) {
    if (id === "1") 
      return "Genes with hypothetical upstream TF" ;
    else if (id === "2")
      return "Genes without hypothetical upstream TF";
    else
      return "";
  }
}
```