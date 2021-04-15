# Evolutionary conservation of human genes（千葉・池田）(mode対応版)

## Endpoint

https://orth.dbcls.jp/sparql-dev

## Parameters
* `categoryIds` (type: branch)
  * example: 1,2,3
* `queryIds` (type: ncbigene)
  * example: 100,101,102
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

## `input_branch`
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
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX orth: <http://purl.org/net/orth#>
PREFIX homologene: <https://ncbi.nlm.nih.gov/homologene/>
PREFIX taxid: <http://identifiers.org/taxonomy/>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>
PREFIX hop: <http://purl.org/net/orthordf/hOP/ontology#>

{{#if mode}}
SELECT DISTINCT ?human_gene ?branch ?branch_id ?branch_label
{{else}}
SELECT ?branch_id ?branch_label (COUNT(?human_gene) AS ?gene_count)
{{/if}}
WHERE {
  {{#if input_branch}}
  VALUES ?branch_id { {{#each input_branch}} {{this}} {{/each}} }
  {{/if}}
  ?branch a hop:HomoloGeneBranch ;
      dct:identifier ?branch_id ;
      rdfs:label ?branch_label ;
      hop:timeMya ?branch_time_mya .
  {
    SELECT ?human_gene (max(?time) as ?branch_time_mya)
    WHERE {
      {{#if input_genes}}
      VALUES ?human_gene { {{#each input_genes}} ncbigene:{{this}} {{/each}} }
      {{/if}}
      ?grp orth:inDataset homologene: ;
          orth:hasHomologousMember ?human_gene, ?gene .
      ?human_gene orth:taxon taxid:9606 .
      ?gene orth:taxon/hop:branch/hop:timeMya ?time .
    }
  }
}
{{#unless mode}}
ORDER BY ?branch_id
{{/unless}}
```

## `return`

```javascript
({ main, mode }) => {
  if (mode === "idList") {
    return Array.from(new Set(
      main.results.bindings.map((elem) =>
        elem.human_gene.value.replace("http://identifiers.org/ncbigene/", "")
      )
    ));
  } else if (mode === "objectList") {
    return main.results.bindings.map((elem) => ({
      id: elem.human_gene.value.replace("http://identifiers.org/ncbigene/", ""),
      attribute: {
        categoryId: elem.branch_id.value,
        uri: elem.branch.value,
        label: elem.branch_label.value
      }
    }));
  } else {
    return main.results.bindings.map((elem) => ({
      categoryId: elem.branch_id.value,
      label: elem.branch_label.value,
      count: Number(elem.gene_count.value)
    }));
  }
};
```