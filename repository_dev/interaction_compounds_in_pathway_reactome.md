# chebi reactome pathway（守屋）

- ChEBI の Reactome パスウェイの内訳
 - 産物と制御
   - reaction ^biopax:controlled/biopax:controller control-component
   - reaction biopax:left|biopax:right|biopax:product product-component
   
## Description

- Data sources
    - Reactome-classified pathways for each chemical compound
    - This item based on the data of  Reactome Version 71 (02, December 2019).
        - The latest data can be obtained from the URL below. https://reactome.org/download-data
- Query
    - Input
        - Reactome ID , ChEBI ID
    - Output
        - The number of ChEBI entries included in each pathways in Reactome
        - If a ChEBI id is entered, it returns the pathway to which the chemical compound belongs



## Parameters

* `categoryIds` (type: reactome)
  * example: R-HSA-1640170 (Cell Cycle)
* `queryIds` (type: chebi)
  * example: 17232 77008 16728 16208 64116 15753 138249 8050 15440 17780 57926 15611 76931 29127 16650 17826 18252
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
  - 内訳返す場合と遺伝子リスト返す場合を handlebars で条件分岐
  - パスウェイ、ChEMBLリストでのフィルタリング
```sparql
PREFIX biopax: <http://www.biopax.org/release/biopax-level3.owl#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
{{#if mode}}
SELECT DISTINCT ?chebi ?category ?label
{{else}}
SELECT DISTINCT ?category ?label (COUNT (DISTINCT ?chebi) AS ?count) (SAMPLE(?child_path) AS ?child)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/reactome>
WHERE {
  VALUES ?child_path_type { biopax:Pathway biopax:BiochemicalReaction }
{{#if queryArray}}
  VALUES ?chebi { {{#each queryArray}} "CHEBI:{{this}}"^^xsd:string {{/each}} }
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
  ?component biopax:component*/biopax:entityReference/biopax:xref [
    biopax:db "ChEBI"^^xsd:string ;
    biopax:id ?chebi
  ] .
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
  let idVarName = "chebi";
  let idPrefix = "CHEBI:";
  if (mode == "objectList") return data.results.bindings.map(d=>{
    return {
      id: d[idVarName].value.replace(idPrefix, ""),
      attribute: {
        categoryId: d.category.value,
        label: d.label.value
      }
    }
  });
  if (mode == "idList") return data.results.bindings.map(d=>d[idVarName].value.replace(idPrefix, ""));

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