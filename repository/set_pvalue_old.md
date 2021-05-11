# Set p-value - 内訳 SPARQList に p-value 追加 (test)

* queryIds それぞれのカテゴリで enrich してるか p-value を計算 ( Σ combination(n,k) * p^k * (1-p)^(n-k) )
  * DAVID の EASE score に合わせて n=>n-1, k=>k-1 で計算
  * 参考: https://david.ncifcrf.gov/content.jsp?file=functional_annotation.html

## Parameters

* `sparqlet`
  * default: https://integbio.jp/togosite/sparqlist/api/uniprot_keywords_cellular_component
* `categoryIds`
  * example:
* `queryIds`
  * default: Q9NYF8,Q4V339,A6NCE7,A7E2F4,P69849,A6NN73,Q92928,Q5T1J5,P0C7P4,Q6DN03,P09874,Q08211,Q5T4S7,P12270,Q9UPN3,P07814,P53621,P49321,P0C629,Q9BZK8,Q9BY65
* `population` total population (e.g. human uniprot 75,776)
  * default: 75776

  
## `httpreq`

```javascript
async ({sparqlet, categoryIds, queryIds, population})=>{
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
  
  let calcPval = (n, x, p) => {  // n: query total, x: category hit, p: probability; (category total - category hit) / (population - query total)
    let pValue = 0;
    for (let k = x; k <= n; k++) {
      let combination = 1;
      for (let i = 0; i < k; i++) {
        combination *= (n - i) / (k - i);
      }
      pValue += combination * p**k * (1-p)**(n-k);
    }
    return pValue;
  }
  let sparqlSpliter = "https://integbio.jp/togosite/sparqlist/api/sparqlist_splitter";
  
  let body = false;
  if (categoryIds) body = "categoryIds= " + categoryIds;
  
  let categoryTotal = {};
  let res = await fetchReq(sparqlet, body);
  for (let d of res) {
    categoryTotal[d.categoryId] = d.count;
  }
  
  queryIds = queryIds.replace(/,/g, " ");
  let queryArray = queryIds.split(/\s+/);
  if (body) body += "&queryIds=" + queryIds + "&sparqlet=" + sparqlet;
  else body = "queryIds=" + queryIds + "&sparqlet=" + sparqlet;
  res = await fetchReq(sparqlSpliter, body);
  for (let i = 0; i < res.length; i++) {
    if (res[i].count == 1) res[i].pValue = 1;
    else res[i].pValue = calcPval(queryArray.length - 1, res[i].count - 1, (categoryTotal[res[i].categoryId] - res[i].count + 1) / (population - queryArray.length + 1));
    res[i].total = categoryTotal[res[i].categoryId];
  }
  return res;
}
```
  