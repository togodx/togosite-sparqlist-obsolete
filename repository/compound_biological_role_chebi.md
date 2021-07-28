# WIP: ChEBI Biological Role での分類 （川島、建石、信定）

## Description

- Data sources
    -  [Chemical Entities of Biological Interest (ChEBI) ](https://www.ebi.ac.uk/chebi/) 
- Query
    - Input
        - ChEBI id (number)
    - Output
        -  [Biological Role (CHEBI:24432)](https://www.ebi.ac.uk/chebi/searchId.do?chebiId=CHEBI:24432) and its subcategories

## Parameters

* `categoryIds` Biological Role（CHEBI:24432）の下位階層のID（数字のみ）またはIDのリスト
  * default:  
  * example: 39317, 35222
* `queryIds` 化合物のID（数字のみ）またはIDのリスト
  * default: 
  * example: 18012, 27732, 17594, 16866, 46195, 62867   
* `mode` 必須パラメータ。内訳の代わりに該当する ID のリストを返す（デフォルトはオフ）idList: リストだけ、objectList: Biological Roleの入ったリスト（Biological Role　は下階層ではなく、categoryid で指定したカテゴリ）
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
                          ?biological_role                                      
                          GROUP_CONCAT(DISTINCT ?biological_label; SEPARATOR = ", ") AS ?biological_label
{{else}}
SELECT distinct (COUNT (DISTINCT ?compound) AS ?count) ?biological_role  ?biological_label (bound(?x) as ?haschild)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/chebi>
WHERE 
{
  {{#if queryArray}}
    VALUES ?compound { {{#each queryArray}} CHEBI:{{this}} {{/each}} }
  {{/if}}
  {{#if categoryArray}}
     {{#if mode}}
        VALUES ?biological_role  { {{#each categoryArray}} CHEBI:{{this}} {{/each}} }
     {{else}}
        VALUES ?parent { {{#each categoryArray}} CHEBI:{{this}} {{/each}} }
        ?biological_role  rdfs:subClassOf ?parent.  
     {{/if}}
          
  {{else}}
        VALUES ?parent {obo:CHEBI_24432}
        ?biological_role  rdfs:subClassOf ?parent.  
  
  {{/if}}
      
  ?compound a owl:Class ;
    rdfs:label ?label ;
    rdfs:subClassOf ?r .
  ?r a owl:Restriction ;
    owl:onProperty obo:RO_0000087 ;
    owl:someValuesFrom ?role .
  ?role rdfs:subClassOf* ?biological_role  .
  ?biological_role  rdfs:label ?biological_label .
  optional{
  	?x rdfs:subClassOf ?biological_role .
  }	
  
}
{{#unless mode}}

GROUP BY ?biological_label ?biological_role  ?x
ORDER BY DESC(?count)
{{/unless}}

```



## `return`
- 整形
```javascript
({mode, data})=>{
  const idVarName = "compound";
  const idLabelVarName = "label";
  const categoryVarName = "biological_role";
  const categoryLabelVarName = "biological_label";
  
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