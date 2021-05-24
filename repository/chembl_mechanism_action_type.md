# ChEMBLを作用機作で分類する（信定・八塚・鈴木）

## Description

- Data sources
    - (More data sources description goes here..)
    - ChEMBL-RDF: http://ftp.ebi.ac.uk/pub/databases/chembl/ChEMBL-RDF/28.0/
- Query
    - (More query details go here..)
    -  Input
        - ChEMBL ID
    - Output
        - Mechanism action type

## Parameters

* `categoryIds` (type: mechanism_action_type)
  * 以下のIDを指定
  * defintion: "1: INHIBITOR", "2: ANTAGONIST", "3: AGONIST", "4: BINDING AGENT", "5: BLOCKER", "6: MODULATOR", "7: POSITIVE ALLOSTERIC MODULATOR", "8: HYDROLYTIC ENZYME", "9: PARTIAL AGONIST", "10: ACTIVATOR", "11: OTHER", "12: OPENER", "13: CROSS-LINKING AGENT", "14: POSITIVE MODULATOR", "15: SEQUESTERING AGENT", "16: RELEASING AGENT", "17: CHELATING AGENT", "18: NEGATIVE ALLOSTERIC MODULATOR", "19: INVERSE AGONIST", "20: STABILISER", "21: SUBSTRATE", "22: ANTISENSE INHIBITOR", "23: PROTEOLYTIC ENZYME", "24: DISRUPTING AGENT", "25: NEGATIVE MODULATOR", "26: RNAI INHIBITOR", "27: OXIDATIVE ENZYME", "28: REDUCING AGENT", "29: ALLOSTERIC ANTAGONIST", "30: DEGRADER"
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
                  15: "SEQUESTERING AGENT", 16: "RELEASING AGENT", 17: "CHELATING AGENT", 18: "NEGATIVE ALLOSTERIC MODULATOR", 19: "INVERSE AGONIST", 20: "STABILISER", 21: "SUBSTRATE", 22: "ANTISENSE INHIBITOR", 23: "PROTEOLYTIC ENZYME", 24: "DISRUPTING AGENT", 25: "NEGATIVE MODULATOR", 26: "RNAI INHIBITOR", 27: "OXIDATIVE ENZYME", 28: "REDUCING AGENT", 29: "ALLOSTERIC ANTAGONIST", 30: "DEGRADER"}
  categoryIds = categoryIds.replace(/,/g, " ");
  if (categoryIds.match(/[^\s]/)) return categoryIds.split(/\s+/).map(id => {return {'cid': id, 'mec': id2ctg[parseInt(id)]}});
  return Object.keys(id2ctg).map(id => {return {'cid': id, 'mec': id2ctg[parseInt(id)]}});
}
```

## Endpoint

https://integbio.jp/togosite/sparql

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
          label: capitalize(idStrs[1])
        }
      };
    });
  }else{
    return query_mechanism_action_type.results.bindings.map(d=>{
      var idStrs = d.mec_id.value.split(":");
      return {
        categoryId: parseInt(idStrs[0]),
        label: capitalize(idStrs[1]),
        count: Number(d.count.value)
      };
    });
  }
  function capitalize(label) {
    const s = label.split(' ').map((word) => {
      if (word === 'RNAI') {
        return 'RNAi';
      } else {
        return word.toLowerCase();
      }
    }).join(' ');
    return s.charAt(0).toUpperCase() + s.substring(1);
  }
}
```