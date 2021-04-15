# Set color group for metastanza - 内訳 SPARQList で、指定 ID を含むカテゴリにグループ名を付与（守屋）

## Parameters

* `sparqlet`
  * default: https://integbio.jp/togosite/sparqlist/api/uniprot_mass
* `queryId` ID ひとつ
  * default: Q9NYF8
  
## `httpreq`

```javascript
async ({sparqlet, queryId})=>{
  let fetchReq = async (url, body) => {
    let options = {	
      method: 'POST',
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    }
    if (body) options.body = body;
    return await fetch(url, options).then(res=>res.json());
  }
  
  let targetCategoryIds = [];
  let res = await fetchReq(sparqlet, "queryIds=" + queryId);
  for (let d of res) {
    targetCategoryIds.push(d.categoryId);
  }
  res =  await fetchReq(sparqlet);
  for (let i = 0; i < res.length; i++) {
    if (targetCategoryIds.includes(res[i].categoryId)) res[i].group = "Contains " + queryId;
    else res[i].group = "Others";
  }
  return res;
}
```
  