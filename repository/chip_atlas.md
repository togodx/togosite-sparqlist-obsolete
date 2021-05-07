# Genes with hypothetical upstream TFs in ChIP-Atlas（大田・池田・小野・千葉）(mode対応版)

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters
* `categoryIds` (type: 1 or 2)
  * example: 1
* `queryIds` (type: ensemble gene)
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
      return "with hypothetical upstream TF" ;
    else if (id === "2")
      return "without hypothetical upstream TF";
    else
      return "";
  }
}
```