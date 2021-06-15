# WIP: ChEBI Application での分類 

## Description

- Data sources
    -  [Chemical Entities of Biological Interest (ChEBI) ](https://www.ebi.ac.uk/chebi/) 
- Query
    - Input
        - ChEBI id (number)
    - Output
        -  [Application (CHEBI:33232)](https://www.ebi.ac.uk/chebi/searchId.do?chebiId=CHEBI:33232) and its subcategories of Mondo

## Parameters

* `categoryIds` Application（CHEBI:33232）の下位階層のID（数字のみ）またはIDのリスト
  * default:  
  * example: 52217, 64047
* `queryIds` 化合物のID（数字のみ）またはIDのリスト
  * default: 
  * example: 18012, 27732, 17594, 16866, 46195, 62867   
* `mode` 必須パラメータ。内訳の代わりに該当する ID のリストを返す（デフォルトはオフ）idList: リストだけ、objectList: Attributeの入ったリスト（Attribute は下階層ではなく、categoryid で指定したカテゴリ）
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
                          ?application                                      
                          GROUP_CONCAT(DISTINCT ?application_label; SEPARATOR = ", ") AS ?application_label
{{else}}
SELECT distinct (COUNT (DISTINCT ?compound) AS ?count) ?application ?application_label (bound(?x) as ?haschild)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/chebi>
WHERE 
{
  {{#if queryArray}}
    VALUES ?compound { {{#each queryArray}} CHEBI:{{this}} {{/each}} }
  {{/if}}
  {{#if categoryArray}}
     {{#if mode}}
        VALUES ?application { {{#each categoryArray}} CHEBI:{{this}} {{/each}} }
     {{else}}
        VALUES ?parent { {{#each categoryArray}} CHEBI:{{this}} {{/each}} }
        ?application rdfs:subClassOf ?parent.  
     {{/if}}
          
  {{else}}
        VALUES ?parent {obo:CHEBI_33232}
        ?application rdfs:subClassOf ?parent.  
  
  {{/if}}
      
  ?compound a owl:Class ;
    rdfs:label ?label ;
    rdfs:subClassOf ?r .
  ?r a owl:Restriction ;
    owl:onProperty obo:RO_0000087 ;
    owl:someValuesFrom ?role .
  ?role rdfs:subClassOf* ?application .
  ?application rdfs:label ?application_label .

  ?x rdfs:subClassOf ?application.
}
{{#unless mode}}

GROUP BY ?application_label ?application ?x
ORDER BY DESC(?count)
{{/unless}}

```



## `return`
- 整形
```javascript
({mode, data})=>{
  const idVarName = "compound";
  const idLabelVarName = "label";
  const categoryVarName = "application";
  const categoryLabelVarName = "application_label";
  
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