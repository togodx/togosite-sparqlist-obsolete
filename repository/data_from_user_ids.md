# Data from user IDs - 内訳 SPARQList を user IDs でフィルタ (test)

* sparqlet: Property の API (properties.json の 'data')
* primaryKey: Property で対象とするデータベース (properties.json の 'primaryKey')
* categoryIds: 下層を取ってくる場合に使用 (要、UIなどの検討）
* userKey: Upload された ID のデータベース
* userIds: Upload された ID リスト

## Parameters

* `sparqlet`
  * default: https://integbio.jp/togosite/sparqlist/api/refex_specific_high_expression
* `primaryKey` databse of SPARQLet
  * default: ncbigene
* `categoryIds`
  * example:
* `userKey` database of uploaded user IDs
  * default: uniprot
* `userIds` uploaded user IDs
  * default: Q9NYF8,Q4V339,A6NCE7,A7E2F4,P69849,A6NN73,Q92928,Q5T1J5,P0C7P4,Q6DN03,P09874,Q08211,Q5T4S7,P12270,Q9UPN3,P07814,P53621,P49321,P0C629,Q9BZK8,Q9BY65

  
## `httpreq`

```javascript
async ({sparqlet, categoryIds, userIds, userKey, primaryKey})=>{
  let fetchReq = async (url, body) => {
    let options = {	
      method: 'POST',
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    }
    if (body) options.body = body;
    console.log(url);
    console.log(body);
    return await fetch(url, options).then(res=>res.json());
  }
  
  let togoidSparqlistSplitter = "https://integbio.jp/togosite/sparqlist/api/togoid_sparqlist_splitter";  
  let sparqlistSplitter = "https://integbio.jp/togosite/sparqlist/api/sparqlist_splitter";
  let togoidApi = "https://integbio.jp/togosite/sparqlist/api/togoid_route_sparql"; // SPARQList での仮実装 2
  let idLimit = 2000; // split 判定

  // convert user IDs to primary IDs for SPARQLet
  let queryIds = "";
  if (userKey != primaryKey) {
    let body = "source=" + userKey + "&target=" + primaryKey + "&ids=" +  encodeURIComponent(userIds);
    let togoidPair;
    if (queryIds.split(/,/).length <= idLimit) {
      togoidPair = await fetchReq(togoidApi, body);
    } else {
      body += "&sparqlet=" + encodeURIComponent(togoidApi) + "&limit=" + idLimit;
      togoidPair = await fetchReq(togoidSparqlistSplitter, body).map(d=>d.target_id).join(",");
    }
    queryIds = togoidPair.map(d=>d.target_id).join(",");
  } else {
    queryIds = userIds;
  }
  
  if (!queryIds.match(/\w/)) return [];
  
  // get property data
  let body = "queryIds=" + queryIds;
  if (categoryIds) body += "&categoryIds= " + categoryIds;
  if (queryIds.split(/,/).length <= idLimit) return await fetchReq(sparqlet, body);
  body += "&sparqlet=" + encodeURIComponent(sparqlet) + "&limit=" + idLimit;
  return await fetchReq(sparqlistSplitter, body);
}
```
  