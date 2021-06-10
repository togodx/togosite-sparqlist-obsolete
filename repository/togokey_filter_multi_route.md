# togokey filter (aggregate SPARQList)(非同期版)絞り込み条件で検索して、条件にあった togo key リストを返す

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
  const fetchReq = async (url, options, body) => {
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

  const togositeConfig = "https://raw.githubusercontent.com/dbcls/togosite/develop/config/togosite-human/properties.json";
  //const sparqlSplitter = "https://integbio.jp/togosite/sparqlist/api/togoid_sparqlist_splitter";
  //const togoidApi = "https://integbio.jp/togosite/sparqlist/api/togoid_route_sparql";
  const sparqlSplitter = "http://localhost:3000/togosite/sparqlist/api/togoid_sparqlist_splitter";
  const togoidApi = "http://localhost:3000/togosite/sparqlist/api/togoid_route_sparql_multi_route";
  const togositeConfigJson = await fetchReq(togositeConfig, {method: "get"});
  const queryProperties = JSON.parse(properties);
  const queryPropertyIds = queryProperties.map(d => d.propertyId);
  let idLimit = 2000; // split 判定
  if (togoKey == "pubchem_compound") idLimit = 500; // restrict POST respons size
  const start = Date.now(); // debug
     
 // not filter (togoKey = hgnc, uniprot, pdb, mondo)
  const togoidNotFilter = "http://localhost:3000/togosite/sparqlist/api/togokey_not_filter";  
  if (queryPropertyIds.length == 0 && (togoKey == "hgnc" || togoKey == "uniprot" || togoKey == "pdb" || togoKey == "mondo")) {
    if (inputIds) return JSON.parse(inputIds);
    return fetchReq(togoidNotFilter, options, "togoKey=" + togoKey);
  }
  
  const getIdPair = async (configProperty) => {
     const t1 = Date.now() - start; // debug
    
    // get 'primatyKey' ID list by category filtering
    let queryCategoryIds = "";
    for (let queryProperty of queryProperties) {
      if (queryProperty.propertyId == configProperty.propertyId) {
        queryCategoryIds = queryProperty.categoryIds.join(",");
        break;
      }
    }
    configProperty.data = configProperty.data.replace("https://integbio.jp/togosite/sparqlist/", "http://localhost:3000/togosite/sparqlist/");
    let primaryIds = await fetchReq(configProperty.data, options, "mode=idList&categoryIds=" + queryCategoryIds);
    const t2 = Date.now() - start; // debug
    
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
    const t3 = Date.now() - start; // debug
    console.log(configProperty.propertyId + ": start " + t1 + ",mid " + t2 + ",fin " + t3);
    
    return idPair;
  }

  const getAllIdPair = async () => {
    let idPairAll = {};
    for (let configSubject of togositeConfigJson) {
      for (let configProperty of configSubject.properties) {
        if (queryPropertyIds.includes(configProperty.propertyId)) { // クエリに Hit したら
          idPairAll[configProperty.propertyId] = getIdPair(configProperty).then(d => d);
        }
      }
    }
    return idPairAll;
  } 

  let togoId = undefined;
  if (inputIds) {
    togoId = {};
    for (let id of JSON.parse(inputIds)) {
      togoId[id] = true;
    }
  }
  
  let idPairAll = await getAllIdPair();
  // console.log(idPairAll);
  
  for (let configSubject of togositeConfigJson) {
    for (let configProperty of configSubject.properties) {
      if (queryPropertyIds.includes(configProperty.propertyId)) { // クエリに Hit したら  
        let idPair = await idPairAll[configProperty.propertyId];
     
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