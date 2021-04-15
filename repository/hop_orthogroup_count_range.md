# Number of ortholog groups each gene belongs to（千葉・池田）(mode,range対応版)

## Endpoint

https://orth.dbcls.jp/sparql-dev

## Parameters
* `categoryIds` (type: range)
  * example: 1-3
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

## `input_range`
```javascript
({ categoryIds }) => {
  categoryIds = categoryIds.replace(/\s/g, "");
  if (categoryIds.match(/\d-/) || categoryIds.match(/-\d/)) {
    let range = { begin: false, end: false };
    if (categoryIds.match(/^[\d\.]+-/))
      range.begin = Number(categoryIds.match(/^([\d\.]+)-/)[1]);
    if (categoryIds.match(/-[\d\.]+$/))
      range.end = Number(categoryIds.match(/-([\d\.]+)$/)[1]);
    return range;
  }
};
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
  {{#if input_range}}
  FILTER (
    {{#if input_range.begin}} 
      ?grp_count >= {{input_range.begin}}
      {{#if input_range.end}} && {{/if}}
    {{/if}}
    {{#if input_range.end}}
      ?grp_count <= {{input_range.end}}
    {{/if}}
  )
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