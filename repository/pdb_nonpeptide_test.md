# PDBエントリに含まれる非タンパク質で分類(ヒトのみ)（井手）(テスト用4/19)

## Parameters

* `categoryId`
  * example: water, ZINC_ION, SULFATE_ION, GLYCEROL, 1,2-ETHANEDIOL, MAGNESIUM_ION, CHLORIDE_ION, CALCIUM_ION, SODIUM_ION, 2-acetamido-2-deoxy-beta-D-glucopyranose
* `queryIds` (type: PDB)
  * default: 1FJS,1FPC,6BNR
  * example: 1FJS,1FPC,6BNR
* `mode` 
  * example: idList, objectList

## `filter_list`
- Filter 用 PDB を配列に
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## `nonpeptide_list`
- categoryId を配列に
```javascript
({categoryId}) => {
  categoryId = categoryId.replace(" ","_")
  categoryId = categoryId.replace(/,/g," ")
  if (categoryId.match(/[^\s]/)) return categoryId.split(/\s+/);
  return false;
}
```

## Endpoint

https://integbio.jp/togosite/sparql　　　　　　　
- https://integbio.jp/rdf/sparql 開発用に一時的にendopointを変更

## `non_peptide`

```sparql
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX pdbr: <https://rdf.wwpdb.org/pdb/>

{{#if mode}}
SELECT DISTINCT ?PDBentry ?title ?nonpeptide_str ?nonpeptide_name ?nonpeptide_compId #?nonpeptide_label
{{else}}
SELECT ?name (REPLACE(?name, " ","_") AS ?nonpoly_str) ?compId (SUM(?count) AS ?count) {
 {SELECT DISTINCT ?nonpoly_name ?nonpoly_compId COUNT(DISTINCT ?PDBentry) AS ?count
{{/if}}
  FROM <http://rdf.integbio.jp/dataset/togosite/pdbj>
   # FROM <http://rdf.integbio.jp/dataset/pdbj>
    WHERE {
         {{#if filter_list}} VALUES ?PDBentry { {{#each filter_list}} pdbr:{{this}} {{/each}} } {{/if}}
          ?PDBentry   a	  pdbo:datablock .
          ?PDBentry   dc:title  	  ?title .
          ?PDBentry   pdbo:has_pdbx_entity_nonpolyCategory/pdbo:has_pdbx_entity_nonpoly  ?nonpoly .
          ?nonpoly pdbo:pdbx_entity_nonpoly.name     ?nonpoly_name .
          ?nonpoly pdbo:pdbx_entity_nonpoly.comp_id  ?nonpoly_compId .
           
         {{#if nonpeptide_list}}
          VALUES ?nonpoly_query { {{#each nonpeptide_list}} "{{this}}" {{/each}} }
          FILTER CONTAINS(?nonpoly_name, ?nonpoly_query)
         {{/if}}
          {SELECT DISTINCT ?PDBentry {
          ?PDBentry  pdbo:has_entityCategory
               / pdbo:has_entity
               / rdfs:seeAlso <http://identifiers.org/taxonomy/9606> .
          }}
  }}
  BIND(IF(?count < 100, "other", ?nonpoly_name) as ?name)
  BIND(IF(?count < 100, "other", ?nonpoly_compId) as ?compId)
  }
{{#unless mode}}  
GROUP BY ?name ?compId
order by DESC(?count)
#LIMIT 100
{{/unless}} 
```

## `results`

```javascript
({mode, non_peptide})=>{
/*    
    let total = Array.from(new Set(total_count.results.bindings.map(e=>Number(e.count_Num.value)))); 
                                                        //total_countのSPARQLから出てきた数値を"total"に代入
    let non_peptide_value = non_peptide.results.bindings.map(e=>Number(e.count.value));
    let sum_limit = non_peptide_value.reduce((prev,current)=> prev+current,0);   
                                                        //non_peptideのSPARQLでlimit100で取得したCount値の合計を計算     
  　let non_peptide_array = non_peptide.results.bindings;  //non_peptideの結果を配列に入れて最後に"Other"要素を加える
    non_peptide_array.push({"nonpeptide_name": {"type": "literal","value": "Other"},
               "nonpeptide_str":  {"type": "literal","value": "Other" },
               "nonpeptide_compId": { "type": "literal", "value": "Other" },
      "count": {"type": "typed-literal","datatype": "http://www.w3.org/2001/XMLSchema#integer","value": total-sum_limit }});
 */
 
   if (mode == "objectList") return non_peptide.results.bindings.map(d=>{ 
     return {
       id: d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", ""), 
       attribute: {
       categoryId: d.nonpoly_str.value, 
       label: makeLabel(d.name.value, d.compId.value) 
                  }
       };
    });
   if (mode == "idList") return Array.from(new Set(non_peptide.results.bindings.map(d=>d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", "")))); // unique 
   return non_peptide.results.bindings.map(d=>{
     return {
       categoryId: d.nonpoly_str.value, 
       label: makeLabel(d.name.value, d.compId.value), 
       count: Number(d.count.value)
       };
   });
   
   function makeLabel(name, compId) {
    if (name.length > 27) {
      return compId +": "+name.substr(0,27) +"...";
    }  else {
      return name;
    }
  }
}
```

