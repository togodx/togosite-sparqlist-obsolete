# PDBエントリに含まれる非タンパク質で分類(ヒトのみ)（井手）

## Parameters

* `categoryId`
  * example: water, ZINC_ION, SULFATE_ION, GLYCEROL, 1,2-ETHANEDIOL, MAGNESIUM_ION, CHLORIDE_ION, CALCIUM_ION, SODIUM_ION, 2-acetamido-2-deoxy-beta-D-glucopyranose
* `queryIds` (type: PDB)
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

## `nonpoly_list`
- categoryId を配列に
```javascript
({categoryId}) => {
  categoryId = categoryId.replace(/\s+/,"")
  categoryId = categoryId.replace("_"," ")
  if (categoryId) return categoryId.split(/,/);
  return false;
}
```

## Endpoint

https://integbio.jp/togosite/sparql　　　　　　　
- https://integbio.jp/rdf/sparql 開発用に一時的にendopointを変更

## `non_poly`

```sparql
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX pdbr: <https://rdf.wwpdb.org/pdb/>

{{#if mode}}
SELECT DISTINCT ?PDBentry ?title (REPLACE(?nonpoly_name, " ","_") AS ?nonpoly_str) ?nonpoly_name ?nonpoly_compId
{{else}}
SELECT DISTINCT ?nonpoly_name (REPLACE(?nonpoly_name, " ","_") AS ?nonpoly_str) ?nonpoly_compId COUNT(DISTINCT ?PDBentry) AS ?count 
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
         {{#if nonpoly_list}}
          VALUES ?nonpoly_query { {{#each nonpoly_list}} "{{this}}" {{/each}} }
          FILTER CONTAINS(?nonpoly_name, ?nonpoly_query)
         {{/if}}
          {SELECT DISTINCT ?PDBentry {
          ?PDBentry  pdbo:has_entityCategory
               / pdbo:has_entity
               / rdfs:seeAlso <http://identifiers.org/taxonomy/9606> .
          }}
   }
{{#unless mode}}
order by DESC(?count)
LIMIT 100
{{/unless}} 
```

## `total_count`
```sparql
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX pdbr: <https://rdf.wwpdb.org/pdb/>

SELECT DISTINCT COUNT(?PDBentry) AS ?count_Num 
     FROM <http://rdf.integbio.jp/dataset/togosite/pdbj>
   # FROM <http://rdf.integbio.jp/dataset/pdbj>
    WHERE {
         {{#if filter_list}} VALUES ?PDBentry { {{#each filter_list}} pdbr:{{this}} {{/each}} } {{/if}}
          ?PDBentry   a	  pdbo:datablock .
          ?PDBentry   pdbo:has_pdbx_entity_nonpolyCategory
                     /pdbo:has_pdbx_entity_nonpoly
                     /pdbo:pdbx_entity_nonpoly.name     ?nonpoly_name .
         {{#if nonpoly_list}}
          VALUES ?nonpoly_query { {{#each nonpoly_list}} "{{this}}" {{/each}} }
          FILTER CONTAINS(?nonpoly_name, ?nonpoly_query)
         {{/if}}
          {SELECT DISTINCT ?PDBentry {
          ?PDBentry  pdbo:has_entityCategory
               / pdbo:has_entity
               / rdfs:seeAlso <http://identifiers.org/taxonomy/9606> .
          }}
   }
```

## `results`

```javascript
({mode, non_poly ,total_count})=>{
  
   let total = Array.from(new Set(total_count.results.bindings.map(e=>Number(e.count_Num.value)))); 
                                                        //total_countのSPARQLから出てきた数値を"total"に代入
   let non_poly_value = non_poly.results.bindings.map(e=>Number(e.count.value));
   let sum_limit = non_poly_value.reduce((prev,current)=> prev+current,0);   
                                                     //non_polyのSPARQLでlimit100で取得したCount値の合計を計算
   if (total != 0) {
   let non_poly_array = non_poly.results.bindings;  //non_polyの結果を配列に入れて最後に"Other"要素を加える
   non_poly_array.push({"nonpoly_name": {"type": "literal","value": "_other"},
               "nonpoly_str":  {"type": "literal","value": "_other" },
               "nonpoly_compId": { "type": "literal", "value": "_other" },
      "count": {"type": "typed-literal","datatype": "http://www.w3.org/2001/XMLSchema#integer","value": total-sum_limit }});
   };
   //return [total, non_poly_value, sum_limit];
  
   if (mode == "objectList") return non_poly.results.bindings.map(d=>{ 
     return {
       id: d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", ""), 
       attribute: {
       categoryId: d.nonpoly_str.value, 
       label: capitalize(makeLabel(d.nonpoly_name.value, d.nonpoly_compId.value))
                  }
       };
    });
   if (mode == "idList") return Array.from(new Set(non_poly.results.bindings.map(d=>d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", "")))); // unique 
   return non_poly.results.bindings.map(d=>{
     return {
       categoryId: d.nonpoly_str.value, 
       label: capitalize(makeLabel(d.nonpoly_name.value, d.nonpoly_compId.value)),
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
  function capitalize(s) {
    return s.charAt(0).toUpperCase() + s.substring(1).toLowerCase();
  }
}
```

