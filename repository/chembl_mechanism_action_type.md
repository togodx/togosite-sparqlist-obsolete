# ChEMBLを作用機作で分類する（信定・八塚・鈴木）

## Parameters

* `categoryIds` (type: mechanism_action_type)
  * 以下のIDを指定
  * defintion: "1: INHIBITOR", "2: ANTAGONIST", "3: AGONIST", "4: BINDING AGENT", "5: BLOCKER", "6: MODULATOR", "7: POSITIVE ALLOSTERIC MODULATOR", "8: HYDROLYTIC ENZYME", "9: PARTIAL AGONIST", "10: ACTIVATOR", "11: OTHER", "12: OPENER", "13: CROSS-LINKING AGENT", "14: POSITIVE MODULATOR", "15: SEQUESTERING AGENT", "16: SEQUESTERING AGENT", "17: RELEASING AGENT", "18: CHELATING AGENT", "19: NEGATIVE ALLOSTERIC MODULATOR", "20: INVERSE AGONIST", "21: STABILISER", "22: SUBSTRATE", "23: ANTISENSE INHIBITOR", "24: PROTEOLYTIC ENZYME", "25: DISRUPTING AGENT", "26: NEGATIVE MODULATOR", "27: RNAI INHIBITOR", "28: OXIDATIVE ENZYME", "29: REDUCING AGENT", "30: ALLOSTERIC ANTAGONIST", "31: DEGRADER"
  * example: 1, 2
* `queryIds` (type: Chembl ID)
  * example: CHEMBL1071, CHEMBL1200945, CHEMBL1274, CHEMBL204021, CHEMBL2111108, CHEMBL286398, CHEMBL252164, CHEMBL304087, CHEMBL452461, CHEMBL134702, CHEMBL279433, CHEMBL591429, CHEMBL101285, CHEMBL101360, CHEMBL101454, CHEMBL101609, CHEMBL101721, CHEMBL101775, CHEMBL101993, CHEMBL102307
* `mode` (type: string)
  * example: idList or objectList

## `queryArray`
- queryIdsを配列に
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## `categoryArray`
- categoryIdsを配列に
```javascript
({categoryIds})=>{
  const id2ctg = {1: "INHIBITOR", 2: "ANTAGONIST", 3: "AGONIST", 4: "BINDING AGENT", 5: "BLOCKER", 6: "MODULATOR", 7: "POSITIVE ALLOSTERIC MODULATOR",
                  8: "HYDROLYTIC ENZYME", 9: "PARTIAL AGONIST", 10: "ACTIVATOR", 11: "OTHER", 12: "OPENER", 13: "CROSS-LINKING AGENT", 14: "POSITIVE MODULATOR",
                  15: "SEQUESTERING AGENT", 16: "SEQUESTERING AGENT", 17: "RELEASING AGENT", 18: "CHELATING AGENT", 19: "NEGATIVE ALLOSTERIC MODULATOR",
                  20: "INVERSE AGONIST", 21: "STABILISER", 22: "SUBSTRATE", 23: "ANTISENSE INHIBITOR", 24: "PROTEOLYTIC ENZYME", 25: "DISRUPTING AGENT",
                  26: "NEGATIVE MODULATOR", 27: "RNAI INHIBITOR", 28: "OXIDATIVE ENZYME", 29: "REDUCING AGENT", 30: "ALLOSTERIC ANTAGONIST", 31: "DEGRADER"}
  categoryIds = categoryIds.replace(/,/g, " ");
  if (categoryIds.match(/[^\s]/)) return categoryIds.split(/\s+/).map(id => {return {'cid': id, 'mec': id2ctg[parseInt(id)]}});
  return Object.keys(id2ctg).map(id => {return {'cid': id, 'mec': id2ctg[parseInt(id)]}});
}
```

## Endpoint

https://integbio.jp/rdf/mirror/ebi/sparql

## `query_mechanism_action_type`

```sparql
prefix cco: <http://rdf.ebi.ac.uk/terms/chembl#>
{{#if mode}}
SELECT DISTINCT ?chembl_id ?mec_id
{{else}}
SELECT  ?mec_id (COUNT(distinct ?chembl_drug_mechanism) AS ?count)
{{/if}}
WHERE 
{
{{#if queryArray}}
  VALUES ?chembl_id { {{#each queryArray}} "{{this}}" {{/each}} }
{{/if}}
{{#if categoryArray}}
  VALUES (?mec_id ?mechanism_action_type) { {{#each categoryArray}} ("{{this.cid}}:{{this.mec}}" "{{this.mec}}") {{/each}} }
{{/if}}
  ?chembl_drug_mechanism a cco:Mechanism ;
                        cco:hasMolecule ?molecule ;
                        cco:mechanismActionType ?mechanism_action_type .
 ?molecule cco:chemblId ?chembl_id .
}
{{#unless mode}}
group by ?mec_id ?count
order by desc (?count)
{{/unless}}              
```

## `return`

```javascript
({mode, query_mechanism_action_type})=>{
  if (mode == "idList") {
    return query_mechanism_action_type.results.bindings.map(d=> d.chembl_id.value);
  }else if (mode == "objectList") {
    return query_mechanism_action_type.results.bindings.map(d=>{
      var idStrs = d.mec_id.value.split(":");
      return {
        id: d.chembl_id.value,
        attribute: {
          categoryId: parseInt(idStrs[0]),
          label: idStrs[1]
        }
      };
    });
  }else{
    return query_mechanism_action_type.results.bindings.map(d=>{
      var idStrs = d.mec_id.value.split(":");
      return {
        categoryId: parseInt(idStrs[0]),
        label: idStrs[1],
        count: Number(d.count.value)
      };
    });
  }
}
```