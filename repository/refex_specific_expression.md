# RefEx organ specific expression based on ROKU flag 共有 SPARQLet（守屋）

- 組織特異的発現遺伝子の内訳 (default: high)
## Parameters

* `categoryIds` (type: refexo 40 tissue)
  * example: v01_40,v19_40,v32_40 (Cerebrum,Skin,Lang)
* `queryIds` (type: ncbigene)
  * example: 1230,2357,101,2219,374,1880,653,695,366,218,1359,221,722,597,1641,576,2173,1949,2246,1463,1183,273,119,2571,1780,1496,1974,2026,2185,2002,605,491,1024
* `negatively` (低発現)
  * example: 1
* `mode`
  * example: idList, objectList

## Endpoint
https://integbio.jp/togosite/sparql

## `data`
- メイン SPARQL
  - 全体の遺伝子リスト返す（キャッシュが効く）
  - high, low を handlbars で条件分岐

```sparql
PREFIX refexo: <http://purl.jp/bio/01/refexo#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT ?element ?category ?label
FROM <http://rdf.integbio.jp/dataset/togosite/refexo>
FROM <http://rdf.integbio.jp/dataset/togosite/refex_id_relation_human>
FROM <http://rdf.integbio.jp/dataset/togosite/refex_tissue_specific_genechip_human_GSE7307>
FROM <http://rdf.integbio.jp/dataset/togosite/refex_tissue_specific_rnaseq_human_PRJEB2445>
WHERE {
  ?element refexo:affyProbeset ?affy .
{{#if negatively}}
  ?affy refexo:isNegativelySpecificTo ?category .
{{else}}
  ?affy refexo:isPositivelySpecificTo ?category .
{{/if}}
  ?category rdfs:label ?label .
  FILTER (LANG (?label) = "en")
}
```

## `return`
- 組織、遺伝子リストでのフィルタリング、内訳

```javascript
({categoryIds, queryIds, data, mode})=>{
  const idPrefix = "http://identifiers.org/ncbigene/";
  const categoryPrefix = "http://purl.jp/bio/01/refexo#";
  categoryIds = categoryIds.replace(/,/g," ");
  queryIds = queryIds.replace(/,/g," ");
  let filteredList = [];
  let categoryId2label = {};
  let count = {};
  let categoryArray = [];
  let queryArray = [];
  if (categoryIds.match(/[^\s]/)) categoryArray = categoryIds.split(/\s+/);
  if (queryIds.match(/[^\s]/)) queryArray = queryIds.split(/\s+/);
  for (let d of data.results.bindings) {
    let id = d.element.value.replace(idPrefix, "");
    let category = d.category.value.replace(categoryPrefix, "");
    if ((!categoryIds && !queryIds)
        || (categoryIds && !queryIds && categoryArray.includes(category))
        || (!categoryIds && queryIds && queryArray.includes(id))
        || (categoryIds && queryIds && categoryArray.includes(category) && queryArray.includes(id))
      ) {
      filteredList.push({
        id: id,
        attribute: {
          categoryId: category,
          uri: d.category.value,
          label: d.label.value,
        }
      });
      if (!categoryId2label[category]) categoryId2label[category] = d.label.value;
      if (!count[category]) count[category] = 0;
      count[category]++;
    }
  }
  if (mode == "objectList") return filteredList;
  if (mode == "idList") return Array.from(new Set(filteredList.map(d=>d.id)));
  
  return Object.keys(count).sort((a,b)=>{ // sort by category ID
    if (a > b) return 1;
    if (a < b) return -1;
    return 0;
  }).map(id=>{ 
    return {
      categoryId: id,
      label: categoryId2label[id], count:
      count[id]
    }
  });
}
```