# nextprot protein existence level（守屋）

- タンパク質の存在レベルの内訳
- filter type: uniprot

## Parameters

* `categoryIds` (type: neXtProt protein existence level)
  * example: Evidence_at_transcript_level,Predicted
* `queryIds` (type: uniprot)
  * example: Q9NYF8,Q4V339,A6NCE7,A7E2F4,P69849,A6NN73,Q92928,Q5T1J5,P0C7P4,Q6DN03,P09874,Q08211,Q5T4S7,P12270,Q9UPN3,P07814,P53621,P49321,P0C629,Q9BZK8,Q9BY65
* `mode` (要素のリストを返す)
  * example: idList, objectList

## Endpoint 
https://integbio.jp/togosite/sparql

## `data`
- メイン SPARQL
  - 全体のタンパク質リスト返す（キャッシュが効く）
```sparql
PREFIX : <http://nextprot.org/rdf#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX hgnc: <http://purl.uniprot.org/hgnc/>
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX db: <http://purl.uniprot.org/database/>
SELECT DISTINCT ?category ?element
FROM <http://rdf.integbio.jp/dataset/togosite/nextprot>
WHERE {
  ?element ^skos:exactMatch/:existence ?category .
}
```

## `return`
- 存在レベル、タンパク質リストでのフィルタリング、内訳

```javascript
({categoryIds, queryIds, mode, data})=>{
  const idPrefix = "http://purl.uniprot.org/uniprot/";
  const categoryPrefix = "http://nextprot.org/rdf#";
  categoryIds = categoryIds.replace(/,/g," ");
  queryIds = queryIds.replace(/,/g," ");
  let filteredList = [];
  let categoryId2label = {};
  let count = {};
  let categoryArray = [];
  let queryArray = [];
  if (categoryIds.match(/\w/)) categoryArray = categoryIds.split(/\s+/);
  if (queryIds.match(/\w/)) queryArray = queryIds.split(/\s+/);
  for (let d of data.results.bindings) {
    let id = d.element.value.replace(idPrefix, "");
    let category = d.category.value.replace(categoryPrefix, "");
    if ((!categoryIds && !queryIds)
        || (categoryIds && !queryIds && categoryArray.includes(category))
        || (!categoryIds && queryIds && queryArray.includes(id))
        || (categoryIds && queryIds && categoryArray.includes(category) && queryArray.includes(id))
      ) {
      if (!categoryId2label[category]) categoryId2label[category] = category.replace(/_/g, " ");
      filteredList.push({
        id: id,
        attribute: {
          categoryId: category,
          uri: d.category.value,
          label: categoryId2label[category]
        }
      });
      if (!count[category]) count[category] = 0;
      count[category]++;
    }
  }
  if (mode == "objectList") return filteredList;
  if (mode == "idList") return Array.from(new Set(filteredList.map(d=>d.id)));
 
  return Object.keys(count).sort((a, b)=>{
    if (count[a] < count[b]) return 1;
    if (count[a] > count[b]) return -1;
    return 0;
  }).map(id=>{ 
    return {
      categoryId: id, 
      label: categoryId2label[id], 
      count: count[id]
    } 
  });
}
```