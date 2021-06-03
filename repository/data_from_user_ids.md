# Data from user IDs with p-value - 内訳 SPARQList を user IDs でフィルタ

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

## `pValueFlag`
```javascript
({primaryKey})=>{
  let obj = {};
  if (primaryKey == "ncbigene" || primaryKey == "ensembl_gene" || primaryKey == "ensembl_transcript" || primaryKey == "uniprot") {
    let obj = {};
    obj[primaryKey] = true;
    return obj;
  }
  obj.nonPValue = true;
  return obj;
}
```

## Endpoint
https://integbio.jp/togosite/sparql

## `population`
```sparql
PREFIX uniprot: <http://purl.uniprot.org/core/>
PREFIX taxon_up: <http://purl.uniprot.org/taxonomy/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX taxon_id: <http://identifiers.org/taxonomy/>
PREFIX id: <http://identifiers.org/>
SELECT (COUNT (DISTINCT ?entry) AS ?count)
{{#if pValueFlag.ncbigene}}
FROM <http://rdf.ebi.ac.uk/dataset/ensembl/102/homo_sapiens>
WHERE {
   ?entry a id:ncbigene ;
          ^rdfs:seeAlso/obo:RO_0002162	taxon_id:9606 . 
}
{{/if}}
{{#if pValueFlag.ensembl_gene}}
FROM <http://rdf.ebi.ac.uk/dataset/ensembl/102/homo_sapiens>
WHERE {
  ?entry a obo:SO_0001217 ;
         obo:RO_0002162	taxon_id:9606 . 
}
{{/if}}
{{#if pValueFlag.ensembl_transcript}}
FROM <http://rdf.ebi.ac.uk/dataset/ensembl/102/homo_sapiens>
WHERE {
  ?entry a obo:SO_0000234 ;
         obo:RO_0002162	taxon_id:9606 . 
}
{{/if}}
{{#if pValueFlag.uniprot}}
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
  ?entry a uniprot:Protein ;
         uniprot:organism taxon_up:9606 ;
         uniprot:proteome ?proteome .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
}
{{/if}}
{{#if pValueFlag.nonPValue}}
WHERE {
}
{{/if}}  
```


## `distribution`
```javascript
async ({sparqlet, categoryIds, userIds, userKey, primaryKey, pValueFlag, population})=>{
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

 // let togoidSparqlistSplitter = "https://integbio.jp/togosite/sparqlist/api/togoid_sparqlist_splitter";  
 // let sparqlistSplitter = "https://integbio.jp/togosite/sparqlist/api/sparqlist_splitter";
 // let togoidApi = "https://integbio.jp/togosite/sparqlist/api/togoid_route_sparql";
  let togoidSparqlistSplitter = "http://localhost:3000/togosite/sparqlist/api/togoid_sparqlist_splitter";  
  let sparqlistSplitter = "http://localhost:3000/togosite/sparqlist/api/sparqlist_splitter";
  let togoidApi = "http://localhost:3000/togosite/sparqlist/api/togoid_route_sparql";
  let idLimit = 2000; // split 判定
  sparqlet = sparqlet.replace("https://integbio.jp/togosite/sparqlist/", "http://localhost:3000/togosite/sparqlist/");
  
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
  let distribution = [];
  let body = "queryIds=" + queryIds;
  if (categoryIds) body += "&categoryIds= " + categoryIds;
  if (queryIds.split(/,/).length <= idLimit) distribution = await fetchReq(sparqlet, body);
  body += "&sparqlet=" + encodeURIComponent(sparqlet) + "&limit=" + idLimit;
  distribution = await fetchReq(sparqlistSplitter, body);

  // without p-value
  console.log(pValueFlag);
  if (pValueFlag.nonPValue) return distribution;

  // with pvalue (gene, protein)
  let calcPvalue = (a, b, c, d) => {
    let sigDigi = (num, exp) => {
      while (num > 10) {
        num /= 10;
        exp++;
      }
      while (num < 1) {
        num *= 10;
        exp--;
      }
      return [num, exp];
    }
    
    let calcProb = (a, b, c, d) => {
      // prob = num * 10 ** exp;
      let num = 1;
      let exp = 0; 
      for (let i = 1;         i <= a;             i++) { [num, exp] = sigDigi(num / i, exp); }  // 1/a!
      for (let i = b + 1;     i <= a + b;         i++) { [num, exp] = sigDigi(num * i, exp); }  // (a+b)!/b!
      for (let i = c + 1;     i <= a + c;         i++) { [num, exp] = sigDigi(num * i, exp); }  // (a+c)!/c!
      for (let i = d + 1;     i <= c + d;         i++) { [num, exp] = sigDigi(num * i, exp); }  // (c+d)!/d!
      for (let i = b + d + 1; i <= a + b + c + d; i++) { [num, exp] = sigDigi(num / i, exp); }  // (b+d)!/(a+b+c+d)!
      return num * 10 ** exp;
    }
    
    let cutoffProb = calcProb(a, b, c, d);
  
    let max = a + b;
    if (max > a + c) max = a + c;
 
    let pValue = 0;
    for (let i = 0; i <= max; i++) {
      let delta = a - i;
      let tmpProb = 1;
      if (b + delta >= 0 && c + delta >= 0 && d - delta >= 0) tmpProb = calcProb(i, b + delta, c + delta, d - delta);
      if (tmpProb <= cutoffProb) pValue += tmpProb;
    }
    if (pValue > 0.9999) pValue = 1; // 有効数字このくらい？
    return pValue;
  }
  
  population = Number(population.results.bindings[0].count.value);
  let queries = queryIds.split(/,/).length;
  
  // category count
  body = false;
  if (categoryIds) body = "categoryIds= " + categoryIds;
  let categoryTotal = {};
  let res = await fetchReq(sparqlet, body);
  for (let d of res) {
    categoryTotal[d.categoryId] = d.count;
  }
  
  for (let i = 0; i < distribution.length; i++) {
    if (distribution[i].count == 1) distribution[i].pValue = 1;
    else distribution[i].pValue = calcPvalue(distribution[i].count - 1, queries - (distribution[i].count - 1), categoryTotal[distribution[i].categoryId] - (distribution[i].count - 1), population - categoryTotal[distribution[i].categoryId] - queries);
    distribution[i].total = categoryTotal[res[i].categoryId];
  }
  return distribution;
}
```