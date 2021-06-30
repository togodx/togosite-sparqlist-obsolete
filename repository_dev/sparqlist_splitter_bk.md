# SPARQList splitter for togosite attribut bar

* 要素のリストを 'limit' で分割して 'sparqlet' に 'categoryIds' 付きで投げて、各結果を整形して返す
  * order は保持されない（order 初期値はフロントエンドが持っている）
  * ここの SPARQList インターフェースでは数千のリストを扱えないが、POST でここを叩く場合は問題ない
  * 子 SPARQLet が同一サーバーでの場合 localhost を指せたら早い

## Parameters

* `sparqlet`
  * example: https://integbio.jp/togosite/sparqlist/api/uniprot_keywords
* `categoryIds`
  * example: 9992
* `queryIds`
  * example: Q9NU53,Q9H3N8,Q9UBY5,Q9BQP9,Q9NWH7,Q9ULQ1,O43603,Q15758,A0A075B6T7,Q6UW88,Q8N436,Q8NGJ4,P14151,P10153,P11912,P40933,Q96FV3,Q99884,P35247,Q99542,Q8NGG7,Q9P244,Q9BQB4,Q5VTJ3,Q8IV08,Q14697,Q8TDW0,Q8NBI3,A0A0B4J276,P11233,Q8NFZ6,Q9BRR3,Q8IZF6,Q8NGL0,Q9BQ51,P08253,P12319,Q15375,Q8IU80,Q8NG81,A6NJZ3,P25024,P14770,A0A0B4J241,Q8NGC5,P21754,Q8IWB1,Q02742,Q99999,Q8NH50,O60704,P22105,Q6NUQ4,A0A539,Q9Y5K2,Q8N146,Q96Q04,Q7Z3B1,O60755,O00592,P21579,Q8NHK3,C9J442,P26012,P84095,O94907
* `mode`
  * example: idList, objectList
* `limit`
  * default: 2000

## `httpreq`

```javascript
async ({sparqlet, queryIds, categoryIds, mode, limit})=>{
  let fetchReq = async function(url, body){
    let options = {
      method: 'POST',
      headers:	 {
        'Accept': 'application/json',
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    }
    if (body) options.body = body;
    console.log(body);
    return await fetch(url, options).then(res=>res.json());
  }
  queryIds = queryIds.replace(/,/g, " ");
  limit = Number(limit);
  if (queryIds.match(/[^\s]/)) {
    let queryArray = queryIds.split(/\s+/);
    let size = queryArray.length;
    let res = undefined;
    for (let i = 0; i * limit <= size; i++){
      let start = i * limit;
      let end = (i + 1) * limit;
      if (end > size) end = size;
      let body = "queryIds=" + queryArray.slice(start, end).join(",");
      if (categoryIds.match(/[^\s]/)) body += "&categoryIds=" + categoryIds;
      if (mode) body += "&mode=" + mode;
      let json = await fetchReq(sparqlet, body);
      if (res === undefined) { // first
          res = json;
      } else {                 // 2nd or later
        if (mode == "idList" || mode == "objectList") res = res.concat(json);
        else {
          let plus = {};
          for (let d of json) {
            plus[d.categoryId] = d;
          }
          for (let i = 0; i < res.length; i++) {  // add count to previous result
            if (plus[res[i].categoryId]) {
              res[i].count += plus[res[i].categoryId].count;
              delete(plus[res[i].categoryId]);
            }
          }
          if (plus.length > 0) {  // concat remained categories
            res = res.concat(plus);
          }
        }
      }
    }
    if (mode == "idList" || mode == "objectList") return Array.from(new Set(res)); // unique
    return res;
  }
  return await fetchReq(sparqlet, body);
}
```
