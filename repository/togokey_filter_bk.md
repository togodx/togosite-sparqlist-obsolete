# togokey filter (aggregate SPARQList) (togoID SPARQL)

## Parameters

* `togoKey`
  * default: hgnc
* `properties`
  * default: [{"propertyId": "refex_specific_high_expression", "categoryIds": ["v32_40", "v25_40"]}, {"propertyId": "uniprot_keywords_cellular_component","categoryIds": ["472"]}, {"propertyId": "uniprot_pdb_existence", "categoryIds": ["1"]}, {"propertyId": "uniprot_chembl_assay_existence", "categoryIds": ["1"]}]

## `primaryIds`
```javascript
async ({togoKey, properties})=>{
  let fetchReq = async (url, options, body) => {
    if (body) options.body = body;
    //==== debug code
    let res = await fetch(url, options);
    console.log(res.status);
    console.log(url);
    console.log(body);
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

  let togositeConfig = "https://raw.githubusercontent.com/dbcls/togosite/develop/config/togosite.config.json";
  let sparqlSplitter = "https://integbio.jp/togosite/sparqlist/api/test_sparqlist_spltiter_togoid";
  let togoidApi = "https://integbio.jp/togosite/sparqlist/api/test_togoid_alt"; // SPARQList での仮実装
  let togositeConfigJson = await fetchReq(togositeConfig, {method: "get"});
  
  let togoIdArray = [];
  let queryProperties = JSON.parse(properties);
  let queryPropertyIds = queryProperties.map(d => d.propertyId);
  // togosite.config.json で上から
  for (let configSubject of togositeConfigJson) {
    for (let configProperty of configSubject.properties) {
      if (queryPropertyIds.includes(configProperty.propertyId)) { // クエリに Hit したら
        if (configProperty.primaryKey == "pdb" || configProperty.primaryKey == "hp" || configProperty.primaryKey == "nando" || configProperty.primaryKey == "togovar") continue; // TogoID API alt. 未対応

        // get 'primatyKey' ID list by category filtering
        let queryCategoryIds = "";
        for (let queryProperty of queryProperties) {
          if (queryProperty.propertyId == configProperty.propertyId) {
            queryCategoryIds = queryProperty.categoryIds.join(",");
            break;
          }
        }
        let primaryIds = await fetchReq(configProperty.data, options, "mode=idList&categoryIds=" + queryCategoryIds);

        // get 'primalyKey' ID - togoKey' ID list via togoID API
        let idPair = [];
        if (togoKey != configProperty.primaryKey) idPair = await fetchReq(sparqlSplitter, options, "sparqlet=" + encodeURIComponent(togoidApi) + "&source=" + configProperty.primaryKey + "&target=" + togoKey + "&ids=" +  encodeURIComponent(primaryIds.join(" ")));
        else idPair = primaryIds.map(d => {return {source: d, target: d} });
        // console.log(idPair.length + " pairs"); //debug
        
        // set 'togoKey' Ids
        let tmpTogoIdArray = Array.from(new Set(idPair.map(d=>d.target))); // unique array
        if (!togoIdArray.length) togoIdArray = tmpTogoIdArray; // first filtered list
        else {
          for (let togoId of togoIdArray) {
            if (!tmpTogoIdArray.includes(togoId)) togoIdArray = togoIdArray.filter(id => id !== togoId); // remove 'togoKey' ID from list
          }
        }
      }
    }
  }
  return togoIdArray;
}
```
