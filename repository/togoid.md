# togoId alt 2: route 生成して(togoid_route)、pair ごとに SPARQL 叩いて(togoid_bidirectional_pair)、この SPARQList で join

## Parameters
* `source`
  * default: uniprot
* `target`
  * default: hgnc
* `ids`
  * default: Q9D7Q1,Q690N0,Q9NYF8,Q4V339,A6NCE7,A7E2F4,P69849,A6NN73,Q92928,Q5T1J5,P0C7P4,Q6DN03,P09874,Q08211,Q5T4S7,P12270,Q9UPN3,P07814

## `join`
```javascript
async ({source, target, ids})=>{
  let togoidRoute = "https://integbio.jp/togosite/sparqlist/api/togoid_route";
  let togoidPair = "https://integbio.jp/togosite/sparqlist/api/togoid_bidirectional_pair";
  let togoidSplitter = "https://integbio.jp/togosite/sparqlist/api/togoid_sparqlist_splitter";

  let fetchReq = async (url, body)=>{
    let options = {
      method: 'POST',
      headers:	 {
        'Accept': 'application/json',
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    }
    if (body) options.body = body;
    if (body.split(/,/).length > 2000) {
      options.body += "&sparqlet=" + encodeURIComponent(url);
      url = togoidSplitter;
    }
    console.log(body);
    return await fetch(url, options).then(res=>res.json());
  }

  let uniqueArray = (array)=>{
    return Array.from(new Set(array));
  }

  let map = (pair, pair2)=>{
    let medium = {};
    for (let d of pair2) {
      if (!medium[d.source_id]) medium[d.source_id] = [];
      medium[d.source_id].push({id: d.target_id, uri: d.target_uri});
    }
    let pairHash = {};
    for (let d of pair) {
      if (medium[d.target_id]) {
        for (let target of medium[d.target_id]) {
          pairHash[d.source_id + " " + target.id + " " + d.source_uri + " " + target.uri] = true;
        }
      }
    }
    return Object.keys(pairHash).map(d=>{
      let a = d.split(" ");
      return {source_id: a[0], target_id: a[1], source_uri: a[2], target_uri: a[3]}
    })
  }
  
  let route = await fetchReq(togoidRoute, "source=" + source + "&target=" + target);
  
  let pair = await fetchReq(togoidPair, "source=" + route[0] + "&target=" + route[1] + "&ids=" + ids);
  let i = 2;
  while (i < route.length) {
    pair = map(pair, await fetchReq(togoidPair, "source=" + route[i - 1] + "&target=" + route[i] + "&ids=" + uniqueArray(pair.map(d=>d.target_id)).join(",")));
    i++;
  }
  return pair;
}
```