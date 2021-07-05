# WIP: ChEBI Chemical Role での分類 （川島、建石、信定）

## Description

- Data sources
    -  [Chemical Entities of Biological Interest (ChEBI) ](https://www.ebi.ac.uk/chebi/) 
- Query
    - Input
        - ChEBI id (number)
    - Output
        -  [Chemical Role (CHEBI:51086)](https://www.ebi.ac.uk/chebi/searchId.do?chebiId=CHEBI:51086) and its subcategories

## Parameters

* `categoryIds` Chemical Role（CHEBI:51086）の下位階層のID（数字のみ）またはIDのリスト
  * default:  
  * example: 15339, 35223
* `queryIds` 化合物のID（数字のみ）またはIDのリスト
  * default: 
  * example: 18012, 27732, 17594, 16866, 46195, 62867   
* `mode` 必須パラメータ。内訳の代わりに該当する ID のリストを返す（デフォルトはオフ）idList: リストだけ、objectList: Chemical Roleの入ったリスト（Chemical Role は下階層ではなく、categoryid で指定したカテゴリ）
  * example: idList, objectList


## `queryArray`
- ユーザが指定した ID リストを配列に分割

```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
   if (queryIds.match(/[^\s]/))  return queryIds.split(/\s+/);
  return false;
}
```

## `categoryArray`
- ユーザが指定したカテゴリ ID リストを配列に分割

```javascript
({categoryIds}) => {
  categoryIds = categoryIds.replace(/,/g," ");
  if (categoryIds.match(/[^\s]/)) return categoryIds.split(/\s+/);
  return false;
}
```

## Endpoint

https://integbio.jp/togosite/sparql

## `data`
- categoryId があった場合に絞り込み
- queryIds があった場合に絞り込み
```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX CHEBI: <http://purl.obolibrary.org/obo/CHEBI_>

{{#if mode}}
SELECT distinct ?compound GROUP_CONCAT(DISTINCT ?label; SEPARATOR = ", ") as ?label 
                          ?chemical_role                                      
                          GROUP_CONCAT(DISTINCT ?chemical_role_label; SEPARATOR = ", ") AS ?chemical_role_label
{{else}}
SELECT distinct (COUNT (DISTINCT ?compound) AS ?count) ?chemical_role ?chemical_role_label (bound(?x) as ?haschild)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/chebi>
WHERE 
{
  {{#if queryArray}}
    VALUES ?compound { {{#each queryArray}} CHEBI:{{this}} {{/each}} }
  {{/if}}
  {{#if categoryArray}}
     {{#if mode}}
        VALUES ?chemical_role { {{#each categoryArray}} CHEBI:{{this}} {{/each}} }
     {{else}}
        VALUES ?parent { {{#each categoryArray}} CHEBI:{{this}} {{/each}} }
        ?chemical_role rdfs:subClassOf ?parent.  
     {{/if}}
          
  {{else}}
        VALUES ?parent {obo:CHEBI_51086}
        ?chemical_role rdfs:subClassOf ?parent.  
  
  {{/if}}
      
  ?compound a owl:Class ;
    rdfs:label ?label ;
    rdfs:subClassOf ?r .
  ?r a owl:Restriction ;
    owl:onProperty obo:RO_0000087 ;
    owl:someValuesFrom ?role .
  ?role rdfs:subClassOf* ?chemical_role .
  ?chemical_role rdfs:label ?chemical_role_label .
  optional {
    ?x rdfs:subClassOf ?chemical_role .
  }
}
{{#unless mode}}

GROUP BY ?chemical_role_label ?chemical_role ?x
ORDER BY DESC(?count)
{{/unless}}

```



## `return`
- 整形
```javascript
({mode, data})=>{
  const idVarName = "compound";
  const idLabelVarName = "label";
  const categoryVarName = "chemical_role";
  const categoryLabelVarName = "chemical_role_label";
  
  const idPrefix = "http://purl.obolibrary.org/obo/CHEBI_";
  const categoryPrefix = "http://purl.obolibrary.org/obo/CHEBI_";
  
  if (mode == "objectList") return data.results.bindings.map(d=>{
    return {
      id: d[idVarName].value.replace(idPrefix, ""), 
      attribute: {
        categoryId: d[categoryVarName].value.replace(categoryPrefix, ""), 
        uri: d[categoryVarName].value,
        label : d[categoryLabelVarName].value
      }
    }
  });
  if (mode == "idList") return Array.from(new Set(data.results.bindings.map(d=>d[idVarName].value.replace(idPrefix, "")))); // unique
  // mode == NULL
  return data.results.bindings.map(d=>{ 
    return {
      categoryId: d[categoryVarName].value.replace(categoryPrefix, ""), 
      label: d[categoryLabelVarName].value,
      count: Number(d.count.value),
      hasChild: (d.haschild.value)
    };
  });	
}
```