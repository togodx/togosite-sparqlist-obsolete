# Number of ortholog groups each gene belongs to（千葉・池田）(mode対応版)

## Endpoint

https://orth.dbcls.jp/sparql-dev

## Parameters
* `categoryIds` (type: count)
  * example: 1,2,3
* `qureyIds` (type: ncbigene)
  * example: 100,101,102
* `mode`
  * example: idList, objectList

## `input_genes`
```javascript
({ qureyIds }) => {
  qureyIds = qureyIds.replace(/,/g, " ");
  if (qureyIds.match(/\S/)) {
    return qureyIds.split(/\s+/);
  } 
};
```

## `input_counts`
```javascript
({ categoryIds }) => {
  categoryIds = categoryIds.replace(/,/g, " ");
  if (categoryIds.match(/\S/)) {
    return categoryIds.split(/\s+/);
  }
}
```

## `main`

```sparql
PREFIX orth: <http://purl.org/net/orth#>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>

{{#if mode}}
SELECT ?grp_count ?gene
{{else}}
SELECT ?grp_count (COUNT(?grp_count) AS ?count)
{{/if}}
WHERE {
  {
    SELECT ?gene (COUNT(DISTINCT ?grp) AS ?grp_count)
    WHERE {
      {{#if input_genes}}
      VALUES ?gene { {{#each input_genes}} ncbigene:{{this}} {{/each}} }
      {{/if}}
      ?grp orth:inDataset <http://purl.org/net/orthordf/hOP/> ;
          orth:hasHomologousMember ?gene .
    }
  }
  {{#if input_counts}}
  VALUES ?grp_count { {{#each input_counts}} {{this}} {{/each}} }
  {{/if}}
}
{{#unless mode}}
ORDER BY ?grp_count
{{/unless}}
```

## `return`

```javascript
({ main, mode }) => {
  if (mode === "idList") {
    return Array.from(new Set(
      main.results.bindings.map((elem) =>
        elem.gene.value.replace("http://identifiers.org/ncbigene/", "")
      )
    ));
  } else if (mode === "objectList") {
    return main.results.bindings.map((elem) => ({
      id: elem.gene.value.replace("http://identifiers.org/ncbigene/", ""),
      attribute: {
        categoryId: elem.grp_count.value,
        label: elem.grp_count.value
      }
    }));
  } else {
    return main.results.bindings.map((elem) => ({
      categoryId: elem.grp_count.value,
      label: elem.grp_count.value,
      count: elem.count.value
    }));
  }
};
```