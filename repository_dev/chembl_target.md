# ChEMBLでtargetの分類内訳を返す（信定）未完成2

## Parameters

* `categoryIds`
  * example: L1, L2, L3, L4, L5, L6 #空欄はtop
* `queryIds` (type: chembl_target)
  * example: CHEMBL2094255
* `mode`
  * example: idList, objectList

## `top`
- Top レベルかどうかのチェック
```javascript
({categoryIds})=>{
  if (categoryIds == '') return true;
  return false;
}
```

## `categoryArray`
- category ID を配列に分割
```javascript
({categoryIds}) => {
  categoryIds = categoryIds.replace(/,/g," ")
  if (categoryIds.match(/[^\s]/)) return categoryIds.split(/\s+/);
  return false;
}
```

## `queryArray`
- Filter 用 chembl_compoundを配列に
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## Endpoint
https://integbio.jp/togosite/sparql

## `graph`

```sparql
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#>
PREFIX chembl_molecule: <http://rdf.ebi.ac.uk/resource/chembl/molecule/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
SELECT   distinct?label ?level   ?parent_label  ?parent_level 
  FROM <http://rdf.integbio.jp/dataset/togosite/chembl> 
  WHERE
  {
{{#if top}}  
     values ?level { "L1" }
{{else}}
     values ?parent_level { {{#each categoryArray}} "{{this}}" {{/each}} }
 {{/if}}      
     ?targetcomponent a cco:TargetComponent ;
                      cco:organismName ?organism ;
        cco:hasProteinClassification ?class .    
    ?class cco:classLevel ?level ;
           rdfs:label ?label ;
           cco:classPath ?pass ;
           rdfs:subClassOf ?parent .
    ?parent cco:classLevel ?parent_level ;
           rdfs:label ?parent_label .
        FILTER ( ?organism = "Homo sapiens")
    }
```

## `target`

```sparql
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#>
PREFIX chembl_molecule: <http://rdf.ebi.ac.uk/resource/chembl/molecule/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
  {{#if top}}         
  SELECT ?component_type count(distinct?chembl_target)
         FROM <http://rdf.integbio.jp/dataset/togosite/chembl> 
   WHERE
  {
     ?targetcomponent a cco:TargetComponent ;
                      cco:componentType ?component_type ;
                      cco:hasTarget ?chembl_target ;
                      cco:organismName ?organism .
    FILTER ( ?organism = "Homo sapiens")
    }
{{else}}
	{{#if mode}}
  SELECT DISTINCT ?uniprot
    {{else}}
SELECT distinct?label count(?uniprot) 
  FROM <http://rdf.integbio.jp/dataset/togosite/chembl> 
   WHERE
  {
    values ?level { {{#each categoryArray}} "{{this}}" {{/each}} }
     ?targetcomponent a cco:TargetComponent ;
                      cco:componentType ?component_type ;
                      cco:hasTarget ?chembl_target ;
                      cco:organismName ?organism ;
        cco:hasProteinClassification ?class .    
    ?class cco:classLevel ?level ;
           rdfs:label ?label ;
    cco:classPath ?pass .
    ?chembl_target  cco:targetType ?target_type ;
                                skos:exactMatch ?targetcomponent2 .
    ?targetcomponent2 skos:exactMatch ?uniprot .
    FILTER ( ?organism = "Homo sapiens")
    FILTER (?component_type = "PROTEIN")
  }
	{{/if}}
{{/if}}
```
