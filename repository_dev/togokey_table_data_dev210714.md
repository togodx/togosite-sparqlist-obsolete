# togokey table data (aggregate SPARQList)(非同期版)テーブルデータ取得（絞り込み CategoryIds あり、絞り込み無し）

## Parameters

* `togoKey`
  * default: hgnc
* `properties`
  * default: [{"propertyId": "refex_specific_high_expression", "categoryIds": ["v32_40", "v25_40"]}, {"propertyId": "uniprot_keywords_cellular_component","categoryIds": ["472"]}, {"propertyId": "uniprot_pdb_existence", "categoryIds": ["1"]}, {"propertyId": "uniprot_chembl_assay_existence", "categoryIds": ["1"]},{"propertyId": "refex_specific_low_expression"}, {"propertyId": "uniprot_phospho_site"}, {"propertyId": "uniprot_keywords_biological_process"}]
* `queryIds` togoKey 100個程度ずつ
  * default: ["4942","5344","6148", "6265","6344","6677","6735","10593","10718","10876"]

## `primaryIds`
```javascript
async ({togoKey, properties, queryIds})=>{
  const fetchReq = async (url, options, body) => {
    console.log(url + " " + body);  // debug
    if (body) options.body = body;
    return await fetch(url, options).then(res=>res.json());
  }

  let options = {
    method: 'POST',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  
  const togositeConfig = "https://raw.githubusercontent.com/dbcls/togosite/develop/config/togosite-human/properties.json";
  const sparqlSplitter = "https://integbio.jp/togosite/sparqlist/api/sparqlist_splitter";
  const togoidApi = "https://integbio.jp/togosite/sparqlist/api/togoid_route_sparql";
  const labelApi = "https://integbio.jp/togosite_dev/sparqlist/api/togokey_label"; 
  //const sparqlSplitter = "http://localhost:3000/togosite/sparqlist/api/sparqlist_splitter";
  //const togoidApi = "http://localhost:3000/togosite/sparqlist/api/togoid_route_sparql";
  //const labelApi = "http://localhost:3000/togosite/sparqlist/api/togokey_label"; 
  const togositeConfigJson = await fetchReq(togositeConfig, {method: "get"});
  const idLimit = 2000; // split 判定
  const queryProperties = JSON.parse(properties);
  const queryPropertyIds = queryProperties.map(d => d.propertyId);
  const togoIdArray = JSON.parse(queryIds);
  const start = Date.now(); // debug
  
  const getAttributeData = async (configProperty) => {
    const t1 = Date.now() - start; // debug
    
    // get 'togoKey' ID - 'primalyKey' ID list via TogoID API
    let idPair = [];
    if (togoKey != configProperty.primaryKey) idPair = await fetchReq(togoidApi, options, "source=" + togoKey + "&target=" + configProperty.primaryKey + "&ids=" + encodeURIComponent(togoIdArray.join(" ")));
    else idPair = togoIdArray.map(d => {return {source_id: d, target_id: d} });
    let togo2primary = {};
    for (let d of idPair) {
      if (!togo2primary[d.source_id]) togo2primary[d.source_id] = [];
      togo2primary[d.source_id].push(d.target_id);
    }
    const t2 = Date.now() - start; // debug
    
    // get attributes of 'primaryKey' Ids
    let primaryIds = Array.from(new Set(idPair.map(d=>d.target_id))).join(",");
    let categoryIdsParam = "";
    for (let queryProperty of queryProperties) {
      if (queryProperty.propertyId == configProperty.propertyId) {
        if (queryProperty.categoryIds) categoryIdsParam = "&categoryIds=" + queryProperty.categoryIds.join(",");
        break;
      }
    }
    let body = "mode=objectList&queryIds=" + encodeURIComponent(primaryIds) + categoryIdsParam;
    let objectList = [];
    // configProperty.data = configProperty.data.replace("https://integbio.jp/togosite/sparqlist/", "http://localhost:3000/togosite/sparqlist/");
    configProperty.data = configProperty.data.replace("https://integbio.jp/togosite/sparqlist/", "https://integbio.jp/togosite_dev/sparqlist/");
    if (primaryIds.length <= idLimit) {
      objectList = await fetchReq(configProperty.data, options, body);
    } else {
      body += "&sparqlet=" + encodeURIComponent(configProperty.data) + "&limit=" + idLimit;
      objectList =  await fetchReq(sparqlSplitter, options, body);
    }  
    const t3 = Date.now() - start; // debug
    console.log(configProperty.propertyId + ": start " + t1 + ",mid " + t2 + ",fin " + t3);
    
    return {"pair": togo2primary, "list": objectList};
  }
  
  const getAllAttributeData = async () => {
    let attributeData = {};
    for (let configSubject of togositeConfigJson) {
      for (let configProperty of configSubject.properties) {
        if (queryPropertyIds.includes(configProperty.propertyId)) { // クエリに Hit したら
          attributeData[configProperty.propertyId] = getAttributeData(configProperty).then(d => d);
        }
      }
    }
    return attributeData;
  } 
 
  let tableData = {};
  for (let togoId of togoIdArray) {
    tableData[togoId] = [];
  }
  
  // label 取得（labelApi が対応しない、ラベルの無い togovar などを入れる場合は注意）
  let togoIdToLabelFetch = fetchReq(labelApi, options, "togoKey=" + togoKey + "&queryIds=" + queryIds); // promise
    
  let attributeDataAll = await getAllAttributeData(); // promise
  // console.log(attributeDataAll);
  
  for (let configSubject of togositeConfigJson) {
    for (let configProperty of configSubject.properties) {
      if (queryPropertyIds.includes(configProperty.propertyId)) { // クエリに Hit したら
        
        let attributeData = await attributeDataAll[configProperty.propertyId];
       // console.log(attributeData);
        let togo2primary = attributeData.pair;
	    let objectList = attributeData.list;
          
        // mapping to 'togoKey' ID
        let primaryId2attribute = {};
        for (let d of objectList) {
          if (!primaryId2attribute[d.id]) primaryId2attribute[d.id] = [];
          primaryId2attribute[d.id].push(d);
        }
        for (let togoId of Object.keys(togo2primary)) {
          let attributeList = [];
          for (let promaryId of togo2primary[togoId]) {
            if (primaryId2attribute[promaryId]) attributeList = attributeList.concat(primaryId2attribute[promaryId]);
          }
          if (attributeList[0]) { 
            tableData[togoId].push({
              propertyId: configProperty.propertyId,
              propertyLabel: configProperty.label,
              propertyKey: configProperty.primaryKey,
              attributes: attributeList
            })
          }
        }
        
      }
    }
  }
  
  // object to list
  let togoIdToLabel = await togoIdToLabelFetch;
  return togoIdArray.map(togoId=>{
    let obj = { id: togoId };
    if (togoIdToLabel[togoId]) obj.label = togoIdToLabel[togoId];
    obj.properties = tableData[togoId];
    return obj;
  })
}
```