# ChEMBLをsubstancetypeで分類する（信定・鈴木・八塚）

## Description

- Data sources
    - (More data sources description goes here..)
    - ChEMBL-RDF: http://ftp.ebi.ac.uk/pub/databases/chembl/ChEMBL-RDF/28.0/
- Query
    - (More query details go here..)
    -  Input
        - ChEMBL ID
    - Output
        - Substance type

## Parameters

* `categoryIds` (type: substance_type)
  * 以下のIDを指定
  * definition: 1:Antibody, 2:Cell, 3:Enzyme, 4:Gene, 5:Oligonucleotide, 6:Oligosaccharide, 7:Protein, 8:Small molecule, 9:Unclassified, 10:Unknown
  * example: 1, 2
* `queryIds` (type: chembl_compound)
  * example: CHEMBL2115492,CHEMBL1071,CHEMBL1200945,CHEMBL1274,CHEMBL204021,CHEMBL2111108,CHEMBL286398,CHEMBL252164,CHEMBL304087,CHEMBL452461,CHEMBL134702,CHEMBL279433,CHEMBL591429,CHEMBL101285,CHEMBL101360
* `mode`
  * example: idList, objectList

## `query_array`
- Filter 用 chembl_compoundを配列に
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## `category_array`
- substance_type を配列に
```javascript
({categoryIds})=>{
  const id2ctg = {1: "Antibody", 2: "Cell", 3: "Enzyme", 4: "Gene", 5: "Oligonucleotide",
                  6: "Oligosaccharide", 7: "Protein", 8: "Small molecule", 9: "Unclassified", 10: "Unknown"}
  categoryIds = categoryIds.replace(/,/g, " ");
  if (categoryIds.match(/[^\s]/)) return categoryIds.split(/\s+/).map(id => {return {'cid': id, 'type': id2ctg[parseInt(id)]}});
  return Object.keys(id2ctg).map(id => {return {'cid': id, 'type': id2ctg[parseInt(id)]}});
}
```

## Endpoint

https://integbio.jp/togosite/sparql

## `count_substance_type`
- SPARQL
  - 内訳返す場合とchembl compoundリストを返す場合を handlbars で条件分岐
  - substance type、chembl compoundリストでのフィルタリング

```sparql
prefix cco: <http://rdf.ebi.ac.uk/terms/chembl#>
{{#if mode}}
SELECT DISTINCT ?chembl_id ?substancetype ?type_id
{{else}}
SELECT  ?type_id (COUNT(distinct ?substance) AS ?count)
{{/if}}
WHERE 
{
{{#if query_array}}
  VALUES ?chembl_id { {{#each query_array}} "{{this}}" {{/each}} }
{{/if}}
{{#if category_array}}
  VALUES (?type_id ?substancetype) { {{#each category_array}} ("{{this.cid}}:{{this.type}}" "{{this.type}}") {{/each}} }
{{/if}}
  ?substance cco:chemblId ?chembl_id ;
            cco:substanceType ?substancetype .
}
{{#unless mode}}
group by ?type_id ?count
ORDER BY Desc(?count)
{{/unless}}
```

## `return`

```javascript
({mode, count_substance_type})=>{
  if (mode == "idList"){
    return count_substance_type.results.bindings.map(d=> d.chembl_id.value);
  }else if (mode == "objectList") {
    return count_substance_type.results.bindings.map(d=>{
      var idStrs = d.type_id.value.split(":");
      return {
        id: d.chembl_id.value,
        attribute: {
          categoryId: parseInt(idStrs[0]),
          label: idStrs[1]
        }
      };
    });
  }else{
    return count_substance_type.results.bindings.map(d=>{
      var idStrs = d.type_id.value.split(":");
      return {
        categoryId: parseInt(idStrs[0]),
        label: idStrs[1],
        count: Number(d.count.value)
      };
    }); 
  }
}
```