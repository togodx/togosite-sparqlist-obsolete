# Glycan subsumption （山本・池田）

## Description

- Data sources
    - [GlyCosmos](https://glycosmos.org/data)

- Query
    - Input
        - GlyTouCan ID
    - Output
        - 

## Endpoint

https://ts.glycosmos.org/sparql

## Parameters
* `categoryIds`
  * example: Base_composition_with_linkage, Monosaccharide_composition_with_linkage, Glycosidic_topology, Linkage_defined_saccharide
* `queryIds` (type: GlyTouCan ID)
  * example: G14606UO,G03652TR,G59522PY,G90484GL
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
PREFIX glycan: <http://purl.jp/bio/12/glyco/glycan#>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX sbsmpt: <http://www.glycoinfo.org/glyco/owl/relation#>
PREFIX structures: <https://glytoucan.org/Structures/Glycans/>
{{#if mode}}
SELECT DISTINCT ?type ?glytoucan
{{else}}
SELECT ?type (count(DISTINCT ?wurcs) as ?count) 
{{/if}}
FROM <http://rdf.glytoucan.org/partner/glycome-db>
FROM <http://rdf.glytoucan.org/partner/bcsdb>
FROM <http://rdf.glytoucan.org/partner/glycoepitope>
FROM <http://rdf.glycosmos.org/glycans/subsumption>
WHERE {
  SELECT DISTINCT ?wurcs ?type
  WHERE {
  {{#if input_queries}}
    VALUES ?glytoucan { {{#each input_queries}} structures:{{this}} {{/each}} }
  {{/if}}
  {{#if input_categories}}
    VALUES ?tissue { {{#each input_categories}} obo:{{this}} {{/each}} }
  {{/if}}
    ?wurcs a ?type ;
           rdfs:seeAlso ?glytoucan ;
           sbsmpt:subsumes* / dcterms:source / glycan:is_from_source / rdfs:seeAlso ?taxonomy .
    VALUES ?taxonomy { <http://identifiers.org/taxonomy/9606> }
  }
}
{{#unless mode}}
GROUP BY ?type
{{/unless}}
```

## `return`

```javascript
({ main, mode }) => {
  if (mode === "idList") {
    return main.results.bindings.map((elem) =>
        elem.glytoucan.value.replace("https://glytoucan.org/Structures/Glycans/", "")
      );
  } else if (mode === "objectList") {
    return main.results.bindings.map((elem) => ({
      id: elem.glytoucan.value.replace("https://glytoucan.org/Structures/Glycans/", ""),
      attribute: {
        categoryId: elem.type.value.replace("http://www.glycoinfo.org/glyco/owl/relation#", ""),
        uri: elem.type.value,
        label: elem.type.value.replace("http://www.glycoinfo.org/glyco/owl/relation#", "")
      }
    });
  } else {
    return main.results.bindings.map((elem) => ({
      categoryId: elem.type.value.replace("http://www.glycoinfo.org/glyco/owl/relation#", ""),
      label: elem.type.value.replace("http://www.glycoinfo.org/glyco/owl/relation#", ""),,
      count: Number(elem.count.value),
    });
  }
}
```