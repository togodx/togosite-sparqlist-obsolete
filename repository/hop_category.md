# Evolutionary conservation patterns of human genes（千葉・池田）(mode対応版)

## Endpoint

https://orth.dbcls.jp/sparql-dev

## Parameters
* `categoryIds` (type: branch)
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
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>
PREFIX orth: <http://purl.org/net/orth#>
PREFIX hop: <http://purl.org/net/orthordf/hOP/ontology#>
PREFIX branch: <http://purl.org/net/orthordf/hOP/branch/>

{{#if mode}}
SELECT DISTINCT ?branch_id ?branch ?branch_label ?gene
{{else}}
SELECT ?branch_id ?branch ?branch_label (COUNT(DISTINCT ?gene) AS ?count)
{{/if}}
WHERE {
  {
    SELECT ?gene ?branch ?branch_label
    WHERE {
      {
        SELECT ?grp ?gene (max(?org_id) as ?max_id)
        WHERE {
          {{#if input_genes}}
          VALUES ?gene { {{#each input_genes}} ncbigene:{{this}} {{/each}} }
          {{/if}}
           ?grp orth:inDataset <http://purl.org/net/orthordf/hOP/> ;
              orth:hasHomologousMember ?gene ;
              orth:organism ?org .
          ?org dct:identifier ?org_id .
        }
      }  
      BIND(URI(CONCAT("http://purl.org/net/orthordf/hOP/organism/", ?max_id)) AS ?max_org)
      ?max_org hop:branch ?branch .
      ?branch a hop:Branch ;
         rdfs:label ?branch_label .
    }
  }
  ?branch dct:identifier ?branch_id .
  {{#if input_branch}}
  VALUES ?branch { {{#each input_branch}} branch:{{this}} {{/each}} }
  {{/if}}
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
        elem.gene.value.replace("http://identifiers.org/ncbigene/", "")
      )
    ));
  } else if (mode === "objectList") {
    return main.results.bindings.map((elem) => ({
      id: elem.gene.value.replace("http://identifiers.org/ncbigene/", ""),
      attribute: {
        categoryId: elem.branch_id.value,
        uri: elem.branch.value,
        label: elem.branch_label.value
      }
    }));
  } else {
    return main.results.bindings.map((elem) => ({
      id: elem.branch_id.value,
      label: elem.branch_label.value,
      count: elem.count.value,
    }));
  }
};