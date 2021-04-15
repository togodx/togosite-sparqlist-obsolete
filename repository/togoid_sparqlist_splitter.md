# SPARQList splitter for togoid

* 要素のリストを 'limit' で分割して 'sparqlet' に 'categoryId' 付きで投げて、各結果を整形して返す
  * order は保持されない（order 初期値はフロントエンドが持っている）
  * ここの SPARQList インターフェースでは数千のリストを扱えないが、POST でここを叩く場合は問題ない
  * 子 SPARQLet が同一サーバーでの場合 localhost を指せたら早い

## Parameters

* `sparqlet`
  * default: https://integbio.jp/togosite/sparqlist/api/togoid_bidirectional_pair
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
    return await fetch(url, options).then(res=>res.json());
  }
/*  let gunzip = await fetch("https://raw.githubusercontent.com/imaya/zlib.js/develop/bin/gunzip.min.js", {method: "get"}).then(res=>res.text());
  eval(gunzip);
  let gunzipIds = new Zlib.Gunzip(ids);
  ids = gunzipIds.decompress(); */
  
  ids = ids.replace(/,/g, " ");
  if (ids.match(/[^\s]/)) {
    let query_array = ids.split(/\s+/);
    let res;
    let query_array_tmp = [];
    let flag = true;
    for (let i = 0; i < query_array.length; i++){
      query_array_tmp.push(query_array[i]);
      if (query_array_tmp.length == limit || i == query_array.length - 1) {
        let body = "source=" + source + "&target=" + target + "&ids=" + query_array_tmp.join(",");
        let json = await fetchReq(sparqlet, body);
	    if (!res) {
          res = json;
          if (res[0].id) flag = false;
	    } else {
          if (flag) res = res.concat(json);
          else {
            let plus = {};
            for (let d of json) {
              plus[d.id] = d;
            }
            for (let i = 0; i < res.length; i++) {
              if (plus[res[i].id]) {
		        res[i].count += plus[res[i].id].count;
		        delete(plus[res[i].id]);
	          }
	        }
	        for (let k of Object.keys(plus)) {
	          res.push(plus[k]);
	        }
	      }
        }
	    query_array_tmp = [];
      }
    }
    return res;
  }
}
```
