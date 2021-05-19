# togokey filter (aggregate SPARQList) 絞り込み条件で検索して、条件にあった togo key リストを返す

## Parameters

* `togoKey`
  * default: hgnc
* `properties`
  * default: [{"propertyId": "refex_specific_high_expression", "categoryIds": ["v32_40", "v25_40"]}, {"propertyId": "uniprot_keywords_cellular_component","categoryIds": ["472"]}, {"propertyId": "uniprot_pdb_existence", "categoryIds": ["1"]}, {"propertyId": "uniprot_chembl_assay_existence", "categoryIds": ["1"]}]
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

//  let togositeConfig = "https://raw.githubusercontent.com/dbcls/togosite/develop/config/togosite.config.json";
  let togositeConfig = "https://raw.githubusercontent.com/dbcls/togosite/develop/config/togosite-human/properties.json";
  let sparqlSplitter = "https://integbio.jp/togosite/sparqlist/api/togoid_sparqlist_splitter";
  let togoidApi = "https://integbio.jp/togosite/sparqlist/api/togoid_route_sparql"; // SPARQList での仮実装 2
  let togositeConfigJson = await fetchReq(togositeConfig, {method: "get"});
  let idLimit = 2000; // split 判定
  
  let start = Date.now(); // debug
  
  let togoIdArray = undefined;
  if (inputIds) {
      togoIdArray = JSON.parse(inputIds);
  }
  
  let queryProperties = JSON.parse(properties);
  let queryPropertyIds = queryProperties.map(d => d.propertyId);
  // togosite.config.json で上から
  for (let configSubject of togositeConfigJson) {
    for (let configProperty of configSubject.properties) {
      if (queryPropertyIds.includes(configProperty.propertyId)) { // クエリに Hit したら
//        if (configProperty.primaryKey == "pdb" || configProperty.primaryKey == "hp" || configProperty.primaryKey == "nando" || configProperty.primaryKey == "togovar") continue; // TogoID API alt. 未対応

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
          if (primaryIds.length <= idLimit) idPair = await fetchReq(togoidApi, options, "source=" + configProperty.primaryKey + "&target=" + togoKey + "&ids=" +  encodeURIComponent(primaryIds.join(" ")));
          else idPair = await fetchReq(sparqlSplitter, options, "sparqlet=" + encodeURIComponent(togoidApi) + "&source=" + configProperty.primaryKey + "&target=" + togoKey + "&ids=" +  encodeURIComponent(primaryIds.join(" ")));
        } else {
          idPair = primaryIds.map(d => {return {source_id: d, target_id: d} });
        }
        // console.log(idPair.length + " pairs"); //debug
        console.log((Date.now() - start) + " ms"); //debug
        
        // set 'togoKey' Ids
        let tmpTogoIdArray = Array.from(new Set(idPair.map(d=>d.target_id))); // unique array
        if (togoIdArray === undefined) togoIdArray = tmpTogoIdArray; // first filtered list
        else {
          for (let togoId of togoIdArray) {
            if (!tmpTogoIdArray.includes(togoId)) togoIdArray = togoIdArray.filter(id => id !== togoId); // remove 'togoKey' ID from list
          }
        }
        if (togoIdArray.length == 0) return [];
      }
    }
  }

  return togoIdArray;
}
```
