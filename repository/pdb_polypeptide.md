# PDBエントリをポリペプチドの数で分類

## Parameters

* `categoryId` -(type:エントリーに含まれるポリペプチドの数)
  * example: 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,58,60,61,62,63,64,66,67,68,70,71,73,74,75,76,77,78,79,80,81,82,125
* `queryIds` -(type: PDB)
  * example: 6TIW,6E7C
* `list_flag`
  * example: 1
  
## `filter_list`
- Filter 用 PDB を配列に
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## `peptide_number_list`
- categoryId を配列に
```javascript
({categoryId}) => {
  categoryId = categoryId.replace(/,/g," ")
  if (categoryId.match(/[^\s]/)) return categoryId.split(/\s+/);
  return false;
}
```

## Endpoint

https://integbio.jp/rdf/sparql

## `count_polypeptide`

```sparql
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX pdbo: <https://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX pdbr: <https://rdf.wwpdb.org/pdb/>

{{#if list_flag}}
SELECT DISTINCT COUNT(?polypeptide) AS ?entity_number ?PDBentry ?title
{{else}}
SELECT DISTINCT ?entity_number COUNT(?entity_number) AS ?count 
 WHERE{
   {{#if peptide_number_list}}
    VALUES ?number { {{#each peptide_number_list}} {{this}} {{/each}} }
    FILTER(?entity_number IN (?number))
    {
   {{/if}}
  SELECT DISTINCT COUNT(?polypeptide) AS ?entity_number ?PDBentry 
{{/if}}
WHERE {
 {{#if filter_list}}
  VALUES ?PDBentry { {{#each filter_list}} pdbr:{{this}} {{/each}} }
 {{/if}}
          ?PDBentry            rdf:type	                      pdbo:datablock .
          ?PDBentry            dc:title  	                  ?title .
          ?PDBentry            pdbo:has_entity_polyCategory   ?polypeptideEntity .
          ?polypeptideEntity   pdbo:has_entity_poly           ?polypeptide .
          ?polypeptide        rdfs:seeAlso                   ?uniprot_link .  #DNAのentryを排除
 }
{{#if peptide_number_list}} } {{/if}}                     
{{#unless list_flag}}                      
}
order by (?entity_number)  
{{/unless}}
```

## `result`

```javascript
({list_flag, count_polypeptide})=>{
   if (list_flag) return count_polypeptide.results.bindings.map(d=>{ 
     return {
       categoryId: d.PDBentry.value.replace("https://rdf.wwpdb.org/pdb/", ""), 
       label: d.title.value, 
       count: Number(d.entity_number.value)
       };
    });
   var id_val = 0;
   return count_polypeptide.results.bindings.map(d=>{ 
     return {
       categoryId: id_val++, 
       label: d.entity_number.value + " peptide(s)", 
       count: Number(d.count.value)
       };
   });	
}
```