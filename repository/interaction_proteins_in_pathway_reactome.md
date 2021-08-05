# UniProt IDs in Reactome pathway（守屋）

## Description

- Data sources
    - Reactome-classified pathways for each protein
    - This item based on the data of Reactome Version 71 (02, December 2019).
       - The latest data can be obtained from the URL below. https://reactome.org/download-data
- Query
    - Input
        - Reactome ID, Uniprot ID
    - Output
        - The number of Uniprot entries included in each pathway in Reactome
        - If a Uniprot id is entered, it returns the pathway to which the protein belongs

## Parameters

* `categoryIds` (type: reactome)
  * example: R-HSA-1640170 (Cell Cycle)
* `queryIds` (type: uniprot)
  * example: Q9NYF8,Q4V339,A6NCE7,A7E2F4,P69849,A6NN73,Q92928,Q5T1J5,P0C7P4,Q6DN03,P09874,Q08211,Q5T4S7,P12270,Q9UPN3,P07814,P53621,P49321,P0C629,Q9BZK8,Q9BY65
* `mode`
  * example: idList, objectList

## `queryArray`
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## `categoryArray`
```javascript
({categoryIds})=>{
  categoryIds = categoryIds.replace(/,/g, " ");
  if (categoryIds.match(/[^\s]/)) return categoryIds.split(/\s+/);
  return false;
}
```

## `rootArray`
- 最上位階層パスウェイリスト
```javascript
({}) => {
  return ["R-HSA-1474244","R-HSA-397014","R-HSA-9612973","R-HSA-1430728","R-HSA-73894","R-HSA-5357801","R-HSA-4839726","R-HSA-8953897","R-HSA-74160","R-HSA-168256","R-HSA-109582","R-HSA-69306","R-HSA-1500931","R-HSA-392499","R-HSA-1266738","R-HSA-1643685","R-HSA-162582","R-HSA-8953854","R-HSA-8963743","R-HSA-1474165","R-HSA-400253","R-HSA-382551","R-HSA-9609507","R-HSA-5653656","R-HSA-1640170","R-HSA-112316","R-HSA-1852241"];
}
```

## Endpoint
https://integbio.jp/togosite/sparql

## `data`
- メイン SPARQL
  - 産物と制御
    - 制御: reaction ^biopax:controlled/biopax:controller control-component
    - 産物: reaction biopax:left|biopax:right|biopax:product product-component
```sparql
PREFIX biopax: <http://www.biopax.org/release/biopax-level3.owl#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
{{#if mode}}
SELECT DISTINCT ?uniprot ?category ?label
{{else}}
SELECT DISTINCT ?category ?label (COUNT (DISTINCT ?uniprot) AS ?count) (SAMPLE(?child_path) AS ?child)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/reactome>
WHERE {
  VALUES ?child_path_type { biopax:Pathway biopax:BiochemicalReaction }
{{#if queryArray}}
  VALUES ?uniprot { {{#each queryArray}} "{{this}}"^^xsd:string {{/each}} }
{{/if}}
{{#if categoryArray}}
  {{#if mode}}
  VALUES ?category { {{#each categoryArray}} "{{this}}"^^xsd:string {{/each}} }
  {{else}}
  VALUES ?parent { {{#each categoryArray}} "{{this}}"^^xsd:string {{/each}} }
  ?parent ^biopax:id/^biopax:xref/biopax:pathwayComponent/biopax:xref/biopax:id ?category .
  {{/if}}
{{else}}
  VALUES ?category { {{#each rootArray}} "{{this}}"^^xsd:string {{/each}} }
{{/if}}
  ?target_path a ?child_path_type ;
        biopax:displayName ?label ;
        biopax:xref [
          biopax:db "Reactome"^^xsd:string ;
          biopax:id ?category ;
        ] ;
        biopax:pathwayComponent* ?reaction .
  ?reaction a biopax:BiochemicalReaction .
  ?reaction biopax:left|biopax:right|biopax:product|^biopax:controlled/biopax:controller ?component .
  # virtuoso bug ?
  #?component biopax:memberPhysicalEntity*/biopax:component*/biopax:entityReference/biopax:xref [
  #  biopax:db "UniProt"^^xsd:string ;
  #  biopax:id ?uniprot
  #] .
  {
    ?component biopax:component*/biopax:entityReference/biopax:xref [
      biopax:db "UniProt"^^xsd:string ;
      biopax:id ?uniprot
    ] .
  } UNION {
    ?component biopax:memberPhysicalEntity/biopax:component*/biopax:entityReference/biopax:xref [
      biopax:db "UniProt"^^xsd:string ;
      biopax:id ?uniprot
    ] .
  }
  OPTIONAL {
    ?target_path biopax:pathwayComponent ?child_path .
    ?child_path a ?child_path_type .
  }
}
{{#unless mode}}
ORDER BY DESC (?count)
{{/unless}}
```

## `return`
- 整形
```javascript
({mode, data})=>{
  let idVarName = "uniprot";
  if (mode == "objectList") return data.results.bindings.map(d=>{
    return {
      id: d[idVarName].value,
      attribute: {
        categoryId: d.category.value,
        label: d.label.value
      }
    }
  });
  if (mode == "idList") return data.results.bindings.map(d=>d[idVarName].value);

  return data.results.bindings.map(d=>{
    let hasChild = false;
    if (d.child) hasChild = true;
    return {
      categoryId: d.category.value, 
      label: d.label.value,
      count: Number(d.count.value),
      hasChild: hasChild
    };
  });	
}
```
