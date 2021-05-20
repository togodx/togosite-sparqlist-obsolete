# UniProt keywords 'Biological process' 内訳（守屋）

- Required SPARQLet: uniprot_keywords

## Parameters

* `categoryIds` (type: UniProt keyword ID) (Req.)
  * default: 9999
* `queryIds` (type: UniProt)
  * example: Q9NYF8,Q4V339,A6NCE7,A7E2F4,P69849,A6NN73,Q92928,Q5T1J5,P0C7P4,Q6DN03,P09874,Q08211,Q5T4S7,P12270,Q9UPN3,P07814,P53621,P49321,P0C629,Q9BZK8,Q9BY65
* `mode`
  * example: idList, objectList
  
## `httpreq`

```javascript
async ({queryIds, categoryIds, mode})=>{
  let url = "https://integbio.jp/togosite/sparqlist/api/uniprot_keywords_wo"; // localhost:port を叩けると早い
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
  