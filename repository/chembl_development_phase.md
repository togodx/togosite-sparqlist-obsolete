# ChEMBLを薬の開発フェーズで分類する（信定）(mode対応版)

## Parameters

* `categoryId` (type: chembl compound development phase)
  * example: 3(efficacy/safety), 2(efficacy)
* `queryIds` (type: chembl_compound)
  * example: CHEMBL1071, CHEMBL1200945, CHEMBL1274, CHEMBL204021, CHEMBL2111108, CHEMBL286398, CHEMBL252164, CHEMBL304087, CHEMBL452461, CHEMBL134702, CHEMBL279433, CHEMBL591429, CHEMBL101285, CHEMBL101360, CHEMBL101454, CHEMBL101609, CHEMBL101721, CHEMBL101775, CHEMBL101993, CHEMBL102307
* `mode`
  * example: idList, objectList

## `queryArray`
- Filter 用 chembl_compoundを配列に
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## `categoryArray`
- development phase を配列に
- 0:no description  1:PK tolerability, 2:Efficacy, 3: Safety & Efficacy, 4:Indication Discovery & expansion
```javascript
({categoryId})=>{
  categoryId = categoryId.replace(/,/g, " ");
  if (categoryId.match(/[^\s]/)) return categoryId.split(/\s+/);
  return false;
}
```

## Endpoint

https://integbio.jp/rdf/mirror/ebi/sparql

## `count_development_phase`
- SPARQL
  - 内訳返す場合とchembl compoundリストを返す場合を handlbars で条件分岐
  - development phase、chembl compoundリストでのフィルタリング

```sparql
prefix cco: <http://rdf.ebi.ac.uk/terms/chembl#>
{{#if mode}}
SELECT DISTINCT ?chembl_id  ?developmentphase
{{else}}
SELECT  ?developmentphase (COUNT(distinct ?chembl_id) AS ?count)
{{/if}}
WHERE 
{
{{#if queryArray}}
  VALUES ?chembl_id { {{#each queryArray}} "{{this}}" {{/each}} }
{{/if}}
{{#if categoryArray}}
  VALUES ?developmentphase { {{#each categoryArray}} {{this}} {{/each}} }
{{/if}}
 ?drug cco:chemblId ?chembl_id ;
           cco:highestDevelopmentPhase ?developmentphase .
  filter not exists { ?drug a cco:DrugIndication }
}
{{#unless mode}}
group by ?developmentphase ?count
ORDER BY Desc(?developmentphase)
{{/unless}}
```

## `return`

```javascript
({count_development_phase, mode})=>{
if (mode === "idList") { 
    return Array.from(new Set(
      count_development_phase.results.bindings.map((elem) =>
       elem.chembl_id.value )                         
    ));
} else if (mode === "objectList") 
{  let label = ["0: No description", "1: PK tolerability", "2: Efficacy", "3: Safety & Efficacy", "4: Indication Discovery & expansion"];
return count_development_phase.results.bindings.map(d=>{ 
    return {
      id:d.chembl_id.value,
      attribute: {
      categoryId: d.developmentphase.value,
      label:  label[parseInt(d.developmentphase.value)],
        }};
     });	
} else 
{ let label = ["0: No description", "1: PK tolerability", "2: Efficacy", "3: Safety & Efficacy", "4: Indication Discovery & expansion"];
return count_development_phase.results.bindings.map(d=>{ 
 return {
      categoryId: d.developmentphase.value,
      label:  label[parseInt(d.developmentphase.value)],
      count: Number(d.count.value)
        };
      });
}}
```