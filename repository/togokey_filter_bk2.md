# togokey filter (aggregate SPARQList)(togoId SPARQL 2)

## Parameters

* `togoKey`
  * default: hgnc
* `properties`
  * default: [{"propertyId": "refex_specific_high_expression", "categoryIds": ["v32_40", "v25_40"]}, {"propertyId": "uniprot_keywords_cellular_component","categoryIds": ["GO_0005886"]}, {"propertyId": "uniprot_pdb_existence", "categoryIds": ["1"]}, {"propertyId": "uniprot_chembl_assay_existence", "categoryIds": ["1"]}]
* `inputIds` Uploaded user IDs
  * example: ["1193","13940","13557","15586","16605","4942","5344","6148", "6265","6344","6677","6735","10593","10718","10876"]
  
## `primaryIds`
```javascript
async ({togoKey, properties, inputIds})=>{
  let fetchReq = async (url, options, body) => {
    if (body) options.body = body;
    //==== debug code
    let res = await fetch(url, options);
    console.log(res.status);
    console.log(url + "?" + body);
    return res.json();
    //====
    // return await fetch(url, options).then(res=>res.json());
  }

  let options = {
    method: 'POST',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }

  let togositeConfig = "https://raw.githubusercontent.com/dbcls/togosite/develop/config/togosite-human/properties.json";
  let sparqlSplitter = "https://integbio.jp/togosite/sparqlist/api/togoid_sparqlist_splitter";
  let togoidApi = "https://integbio.jp/togosite/sparqlist/api/togoid_route_sparql"; // SPARQList 版 ID 変換
  let togositeConfigJson = await fetchReq(togositeConfig, {method: "get"});
  let idLimit = 2000; // split 判定
  
  let start = Date.now(); // debug

  let togoId = undefined;
  if (inputIds) {
    togoId = {};
    for (let id of JSON.parse(inputIds)) {
      togoId[id] = true;
    }
  }
  
  let queryProperties = JSON.parse(properties);
  let queryPropertyIds = queryProperties.map(d => d.propertyId);
  // properties.json を順番に
  for (let configSubject of togositeConfigJson) {
    for (let configProperty of configSubject.properties) {
      if (queryPropertyIds.includes(configProperty.propertyId)) { // クエリに Hit したら

        // get 'primatyKey' ID list by category filtering
        let queryCategoryIds = "";
        for (let queryProperty of queryProperties) {
          if (queryProperty.propertyId == configProperty.propertyId) {
            queryCategoryIds = queryProperty.categoryIds.join(",");
            break;
          }
        }
        let primaryIds = await fetchReq(configProperty.data, options, "mode=idList&categoryIds=" + queryCategoryIds);
        console.log((Date.now() - start) + " ms"); //debug
        
        // get 'primalyKey' ID - togoKey' ID list via togoID API
        let idPair = [];
        if (togoKey != configProperty.primaryKey) {
          let body = "source=" + configProperty.primaryKey + "&target=" + togoKey + "&ids=" +  encodeURIComponent(primaryIds.join(","));
          if (primaryIds.length <= idLimit) {
            idPair = await fetchReq(togoidApi, options, body);
          } else {
            body += "&sparqlet=" + encodeURIComponent(togoidApi) + "&limit=" + idLimit;
            idPair = await fetchReq(sparqlSplitter, options, body);
          }
        } else {
          idPair = primaryIds.map(d => {return {source_id: d, target_id: d} });
        }
        // console.log(idPair.length + " pairs"); //debug
        console.log((Date.now() - start) + " ms"); //debug
        
        // set 'togoKey' Ids
        let filterId = {};
        for (let id of idPair.map(d=>d.target_id)) {
          filterId[id] = true;
        }
        if (togoId === undefined) { // first filtered list
          togoId = filterId;
        } else {
          for (let id of Object.keys(togoId)) { // remove 'togoKey' ID from list
            if (!filterId[id]) delete(togoId[id]);
          }
        }
        if (Object.keys(togoId).length == 0) return [];
      }
    }
  }
  return Object.keys(togoId);
}
```
