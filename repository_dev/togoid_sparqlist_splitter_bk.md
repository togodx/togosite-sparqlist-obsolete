# SPARQList splitter for togoid

* 要素のリストを 'limit' で分割して'sparqlet' (togoid_route_sparql or togoid_binary_pair) に投げる
  * ここの SPARQList インターフェースでは数千のリストを扱えないが、POST でここを叩く場合は問題ない
  * 子 SPARQLet が同一サーバーでの場合 localhost を指せたら早い

## Parameters

* `sparqlet`
  * default: https://integbio.jp/togosite/sparqlist/api/togoid_route_sparql
* `ids`
  * example: Q9NU53,Q9H3N8,Q9UBY5,Q9BQP9,Q9NWH7,Q9ULQ1,O43603,Q15758,A0A075B6T7,Q6UW88,Q8N436,Q8NGJ4,P14151,P10153,P11912,P40933,Q96FV3,Q99884,P35247,Q99542,Q8NGG7,Q9P244,Q9BQB4,Q5VTJ3,Q8IV08,Q14697,Q8TDW0,Q8NBI3,A0A0B4J276,P11233,Q8NFZ6,Q9BRR3,Q8IZF6,Q8NGL0,Q9BQ51,P08253,P12319,Q15375,Q8IU80,Q8NG81,A6NJZ3,P25024,P14770,A0A0B4J241,Q8NGC5,P21754,Q8IWB1,Q02742,Q99999,Q8NH50,O60704,P22105,Q6NUQ4,A0A539,Q9Y5K2,Q8N146,Q96Q04,Q7Z3B1,O60755,O00592,P21579,Q8NHK3,C9J442,P26012,P84095,O94907
* `source`
  * example: uniprot
* `target`
  * example: hgnc
* `limit`
  * default: 2000

## `httpreq`

```javascript
async ({sparqlet, ids, source, target, limit})=>{
  let fetchReq = async function(url, body){
    let options = {
      method: 'POST',
      headers:	 {
        'Accept': 'application/json',
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    }
    if (body) options.body = body;
    console.log(url);
    console.log(body);
    return await fetch(url, options).then(res=>res.json());
  }
  
  let s = Date.now(); // debug code
  ids = ids.replace(/,/g, " ");
  limit = Number(limit);
  if (ids.match(/[^\s]/)) {
    let queryArray = ids.split(/\s+/);
    let res = [];
    let size = queryArray.length;
    for (let i = 0; limit * i <= size; i++){
      let start = i * limit;
      let end = (i + 1) * limit;
      if (end > size) end = size;
      let body = "source=" + source + "&target=" + target + "&ids=" + queryArray.slice(start, end).join(",");
      let json = await fetchReq(sparqlet, body);
      console.log((Date.now() - s) + " ms");
      res = res.concat(json);
    }
    return res;
  }
}
```
