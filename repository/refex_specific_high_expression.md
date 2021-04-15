# RefEx organ specific 'high' expression 内訳（守屋）

- Required SPARQLet: refex_specific_expression

## Parameters

* `categoryIds` (type: refexo 40 tissue)
  * example: v01_40,v19_40,v32_40 (Cerebrum,Skin,Lang)
* `queryIds` (type: ncbigene)
  * example: 1230,2357,101,2219,374,1880,653,695,366,218,1359,221,722,597,1641,576,2173,1949,2246,1463,1183,273,119,2571,1780,1496,1974,2026,2185,2002,605,491,1024
* `mode`
  * example: idList, objectList
  
## `httpreq`

```javascript
async ({queryIds, categoryIds, mode})=>{
  let url = "https://integbio.jp/togosite/sparqlist/api/refex_specific_expression"; // localhost:port を叩けると早い
  let options = {
    method: 'POST',
    body	: "categoryIds=" + categoryIds + "&queryIds=" + encodeURIComponent(queryIds) + "&mode=" + mode,
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  return await fetch(url, options).then(res=>res.json());
}
```
  