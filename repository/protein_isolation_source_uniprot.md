# uniprot isolated tissue（守屋）

## Description

- Data sources
    - [UniProt](https://www.uniprot.org/)

- Query
    - Obtain tissues to which input UniProt entries link with <https://www.uniprot.org/core/isolatedFrom>.
    - Input UniProt entries contain a reference describing the protein sequence obtained from a clone isolated from output tissues.
    - Input
        - UniProt ID
    - Output
        - Tissue
            - [UniProt Controlled vocabulary of tissues](https://www.uniprot.org/docs/tisslist)

## Parameters

* `categoryIds` (type: uniprot keyword ID)
  * example: 1001 (T-cell)
* `queryIds` (type: uniprot)
  * example: Q9NYF8,Q4V339,A6NCE7,A7E2F4,P69849,A6NN73,Q92928,Q5T1J5,P0C7P4,Q6DN03,P09874,Q08211,Q5T4S7,P12270,Q9UPN3,P07814,P53621,P49321,P0C629,Q9BZK8,Q9BY65
* `mode`
  * example: idList, objectList

## `queryArray`
- Query UniProt ID を配列に
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## `categoryArray`
- UniProt keyword ID を配列に
```javascript
({categoryIds})=>{
  categoryIds = categoryIds.replace(/,/g, " ");
  if (categoryIds.match(/[^\s]/)) return categoryIds.split(/\s+/);
  return false;
}
```

## Endpoint
https://integbio.jp/togosite/sparql

## `data`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX tissue: <http://purl.uniprot.org/tissues/>
{{#if mode}}
SELECT DISTINCT ?uniprot ?category ?label
{{else}}
SELECT DISTINCT ?category ?label (COUNT (DISTINCT ?uniprot) AS ?count)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot/tissues>
WHERE {
{{#if queryArray}}
  VALUES ?uniprot { {{#each queryArray}} uniprot:{{this}} {{/each}} }
{{/if}}
{{#if categoryArray}}
  VALUES ?category { {{#each categoryArray}} tissue:{{this}} {{/each}} }
{{/if}}
  ?uniprot a up:Protein ;
          # up:organism taxon:9606 ;
           up:proteome ?proteome ;
           up:isolatedFrom ?category .
  ?category skos:prefLabel ?label .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
}
{{#unless mode}}
ORDER BY DESC (?count)
{{/unless}}
```

## `return`
- 整形
```javascript
({mode, data})=>{
  const idPrfix = "http://purl.uniprot.org/uniprot/";
  const categoryPrefix = "http://purl.uniprot.org/tissues/";
  const idVarName = "uniprot";
  if (mode == "objectList") return data.results.bindings.map(d=>{
    return {
      id: d[idVarName].value.replace(idPrfix, ""), 
      attribute: {
        categoryId: d.category.value.replace(categoryPrefix, ""),
        label: d.label.value
      }
    }
  });
  if (mode == "idList") return data.results.bindings.map(d=>d[idVarName].value.replace(idPrfix, ""));

  return data.results.bindings.map(d=>{
    let hasChild = false;
    if (d.child) hasChild = true;
    return {
      categoryId: d.category.value.replace(categoryPrefix, ""), 
      label: d.label.value,
      count: Number(d.count.value),
      hasChild: hasChild
    };
  });	
}
```