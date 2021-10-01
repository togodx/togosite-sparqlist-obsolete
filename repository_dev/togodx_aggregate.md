# TogoDX aggregate SPARQList (Parameter, JSON 変更) 21.10.01

- 元 togokey_filter

## Parameters

* `togoKey`
  * default: hgnc
* `filters`
  * default: [{"propertyId": "gene_high_level_expression_refex", "categoryIds": ["v32_40", "v25_40"]}, {"propertyId": "protein_cellular_component_uniprot","categoryIds": ["GO_0005886"]}, {"propertyId": "structure_data_existence_uniprot", "categoryIds": ["1"]}, {"propertyId": "interaction_chembl_assay_existence_uniprot", "categoryIds": ["1"]}]
* `inputIds` Uploaded user IDs
  * example: ["1193","13940","13557","15586","16605","4942","5344","6148", "6265","6344","6677","6735","10593","10718","10876"]
  
## `primaryIds`
```javascript
async ({togoKey, filters, inputIds})=>{
  const fetchReq = async (url, options, body) => {
    if (body) options.body = body;
    /* //==== debug code
    let res = await fetch(url, options);
    console.log(res.status);
    console.log(url + "?" + body);
    return res.json();
    //==== */
    return await fetch(url, options).then(res=>res.json());
  }

  let options = {
    method: 'POST',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  
  const togositeConfig = "https://raw.githubusercontent.com/togodx/togodx-config-human/develop/config/properties.json";
  const sparqlSplitter = "togoid_sparqlist_splitter";  // nested SPARQLet relative path
  const togoidApi = "togoid_route_sparql";  // nested SPARQLet relative path
  const togositeConfigJson = await fetchReq(togositeConfig, {method: "get"});
  const queryProperties = JSON.parse(filters);
  let idLimit = 2000; // split 判定
  if (togoKey == "pubchem_compound") idLimit = 500; // restrict POST respons size
  const start = Date.now(); // debug

  // not filter (togoKey = hgnc, uniprot, pdb, mondo)
  const togoidNotFilter = "togokey_not_filter"; // nested SPARQLet relative path
  if (queryProperties.length == 0 && (togoKey == "hgnc" || togoKey == "uniprot" || togoKey == "pdb" || togoKey == "mondo")) {
    if (inputIds && JSON.parse(inputIds)[0]) return JSON.parse(inputIds);
    return fetchReq(togoidNotFilter, options, "togoKey=" + togoKey);
  }
  
  let propertyId2config = {}
  for (let configSubject of togositeConfigJson) {
    for (let configProperty of configSubject.properties) {
      propertyId2config[configProperty.propertyId] = configProperty;
    }
  }

  const getIdPair = async (query) => {
    const propertyId = query.propertyId;
    const config = propertyId2config[propertyId];
     //const t1 = Date.now() - start; // debug
    
    // get 'primatyKey' ID list by category filtering
    let queryCategoryIds = query.categoryIds.join(",");

    config.data = config.data.split(/\//).slice(-1)[0];  // nested SPARQLet relative path
    let primaryIds = await fetchReq(config.data, options, "mode=idList&categoryIds=" + queryCategoryIds);
    //const t2 = Date.now() - start; // debug
    
    // get 'primalyKey' ID - togoKey' ID list via togoID API
    let idPair = [];
    if (togoKey != config.primaryKey) {
      let body = "source=" + config.primaryKey + "&target=" + togoKey + "&ids=" +  encodeURIComponent(primaryIds.join(","));
      if (primaryIds.length <= idLimit) {
        idPair = await fetchReq(togoidApi, options, body);
      } else {
        body += "&sparqlet=" + encodeURIComponent(togoidApi) + "&limit=" + idLimit;
        idPair = await fetchReq(sparqlSplitter, options, body);
      }
    } else {
      idPair = primaryIds.map(d => {return {source_id: d, target_id: d} });
    }
    //const t3 = Date.now() - start; // debug
    //console.log(configProperty.propertyId + ": start " + t1 + ",mid " + t2 + ",fin " + t3);
    
    return idPair;
  }

  const getAllIdPair = async () => {
    let idPairAll = {};
    for (let i = 0; i < queryProperties.length; i++) {
      idPairAll[i] = getIdPair(queryProperties[i]).then(d => d);
    }
    return idPairAll;
  } 

  let togoId = undefined;
  if (inputIds && JSON.parse(inputIds)[0]) {
    togoId = {};
    for (let id of JSON.parse(inputIds)) {
      togoId[id] = true;
    }
  }
  
  let idPairAll = await getAllIdPair();
  // console.log(idPairAll);
  

  for (let i = 0; i < queryProperties.length; i++) {
    const propertyId = queryProperties[i].propertyId;     
    
    const idPair = await idPairAll[i];
     
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
  return Object.keys(togoId);
}
```