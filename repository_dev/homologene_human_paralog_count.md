# Number of human genes in each ortholog group（千葉）(mode対応版)

## Description
- Data sources
  - HomoloGene Release 68: [https://www.ncbi.nlm.nih.gov/homologene/statistics/](https://www.ncbi.nlm.nih.gov/homologene/statistics/)
- Input/Output
  - Input
    - NCBI Gene ID
  - Output
    - Number of paralogs
- Supplementary information
  - The number of duplicated genes (paralogs) in each human gene.
  - ヒトの各遺伝子における重複遺伝子（パラログ）の数です。

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters
* `categoryIds` (type: count)
  * example: 2,3
* `queryIds` (type: ncbigene)
  * example: 101,805,3543,11243
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
  {{#if input_counts}}
  VALUES ?paralog_count { {{#each input_counts}} {{this}} {{/each}} }
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
      count: Number(elem.count.value)
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