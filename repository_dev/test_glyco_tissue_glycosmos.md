# Glycans in tissue （山本・池田）

## Description

- Data sources
    - [GlyCosmos](https://glycosmos.org/data)

- Query
    - Input
        - GlyTouCan ID
    - Output
        - UBERON ID of tissues

## Endpoint

https://ts.glycosmos.org/sparql

## Parameters
* `categoryIds` (type: UBERON ID)
  * example: UBERON_0001155, UBERON_0002107
* `queryIds` (type: GlyTouCan ID)
  * example: 
* `mode` (type: string)
  * example: idList, objectList

## `input_queries`
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
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX info: <http://rdf.glycoinfo.org/glycan/>
PREFIX glycan: <http://purl.jp/bio/12/glyco/glycan#>
PREFIX obo: <http://purl.obolibrary.org/obo/>

{{#if mode}}
SELECT DISTINCT ?tissue_label ?tissue ?glytoucan
{{else}}
SELECT DISTINCT ?tissue_label ?tissue (count(?glytoucan) as ?count) 
{{/if}}
WHERE {
  {{#if input_queries}}
  VALUES ?glytoucan { {{#each input_queries}} info:{{this}} {{/each}} }
  {{/if}}
  {{#if input_categories}}
  VALUES ?tissue { {{#each input_categories}} obo:{{this}} {{/each}} }
  {{/if}}
  VALUES ?taxon { <http://rdf.glycoinfo.org/source/9606> }
  [] glycan:has_glycan ?glytoucan ;
     a <http://purl.jp/bio/4/id/200906013374193296> ;
     glycan:has_taxon ?taxon ;
     glycan:has_tissue ?tissue .
  ?tissue rdfs:label ?tissue_label .
  FILTER(DATATYPE(?tissue_label) = xsd:string)
}
{{#unless mode}}
group by ?tissue ?tissue_label ?taxonomy ?taxon
order by desc(?count)
{{/unless}}
```

## `return`

```javascript
({ main, mode }) => {
  if (mode === "idList") {
    return Array.from(new Set(
      main.results.bindings.map((elem) =>
        elem.glytoucan.value.replace("http://rdf.glycoinfo.org/glycan/", "")
      )
    ));
  } else if (mode === "objectList") {
    return main.results.bindings.map((elem) => ({
      id: elem.glytoucan.value.replace("http://rdf.glycoinfo.org/glycan/", ""),
      attribute: {
        categoryId: elem.tissue.value.replace("http://purl.obolibrary.org/obo/", ""),
        uri: elem.tissue.value,
        label: elem.tissue_label.value
      }
    }));
  } else {
    return main.results.bindings.map((elem) => ({
      categoryId: elem.tissue.value.replace("http://purl.obolibrary.org/obo/", ""),
      label: elem.tissue_label.value,
      count: Number(elem.count.value),
    }));
  }
}
```