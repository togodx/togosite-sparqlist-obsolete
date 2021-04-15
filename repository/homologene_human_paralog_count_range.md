# Number of human genes in each ortholog group（千葉・池田）(mode,range対応版)

## Endpoint

https://orth.dbcls.jp/sparql-dev

## Parameters
* `categoryIds` (type: range)
  * example: 2-3
* `qureyIds` (type: ncbigene)
  * example: 101,805,3543,11243
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
PREFIX homologene: <https://ncbi.nlm.nih.gov/homologene/>
PREFIX taxid: <http://identifiers.org/taxonomy/>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>

{{#if mode}}
SELECT DISTINCT ?paralog_count ?gene
{{else}}
SELECT ?paralog_count (COUNT(DISTINCT ?gene) AS ?count)
{{/if}}
WHERE {
  {{#if input_genes}}
  VALUES ?gene { {{#each input_genes}} ncbigene:{{this}} {{/each}} }
  {{/if}}
  ?grp orth:inDataset homologene: ;
      orth:hasHomologousMember ?gene .
  ?gene orth:taxon taxid:9606 .
  {
    SELECT ?grp (COUNT(DISTINCT ?human_gene) AS ?paralog_count)
    WHERE {
      ?grp orth:inDataset homologene: ;
          orth:hasHomologousMember ?human_gene .
      ?human_gene orth:taxon taxid:9606 .
    }
  }
  {{#if input_range}}
  FILTER (
    {{#if input_range.begin}} 
      ?paralog_count >= {{input_range.begin}}
      {{#if input_range.end}} && {{/if}}
    {{/if}}
    {{#if input_range.end}}
      ?paralog_count <= {{input_range.end}}
    {{/if}}
  )
  {{/if}}
}
{{#unless mode}}
ORDER BY ?paralog_count
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
        categoryId: elem.paralog_count.value,
        label: makeLabel(elem.paralog_count.value)
      }
    }));
  } else {
    return main.results.bindings.map((elem) => ({
      categoryId: elem.paralog_count.value,
      label: makeLabel(elem.paralog_count.value),
      count: elem.count.value
    }));
  }
  
  function makeLabel(paralog_count) {
    if (paralog_count == 1) {
      return "singleton";
    } else if (paralog_count == 2) {
      return "doublet";
    } else if (paralog_count == 3) {
      return "triplet";
    } else {
      return `${paralog_count} genes`;
    }
  }
};
```