# togokey table data (aggregate SPARQList) テーブルデータ取得（絞り込み CategoryIds あり、絞り込み無し）

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
    console.log(body); // debug
    console.log(url);  // debug
    if (body) options.body = body;
    return await fetch(url, options).then(res=>res.json());
  }

  const options = {
    method: 'POST',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  
  const togositeConfig = "https://raw.githubusercontent.com/dbcls/togosite/develop/config/togosite-human/properties.json";
  const sparqlSplitter = "https://integbio.jp/togosite/sparqlist/api/sparqlist_splitter";
  const togoidApi = "https://integbio.jp/togosite/sparqlist/api/togoid_route_sparql";
  const togositeConfigJson = await fetchReq(togositeConfig, {method: "get"});
  const idLimit = 2000; // split 判定
  
  const getAttributeData = async (togoKey, configProperty) => {
     console.log(configProperty.propertyId);  
    // get 'togoKey' ID - 'primalyKey' ID list via TogoID API
    let idPair = [];
    if (togoKey != configProperty.primaryKey) idPair = await fetchReq(togoidApi, options, "source=" + togoKey + "&target=" + configProperty.primaryKey + "&ids=" + encodeURIComponent(togoIdArray.join(" ")));
    else idPair = togoIdArray.map(d => {return {source_id: d, target_id: d} });
    let togo2primary = {};
    for (let d of idPair) {
      if (!togo2primary[d.source_id]) togo2primary[d.source_id] = [];
      togo2primary[d.source_id].push(d.target_id);
    }

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
    if (primaryIds.length <= idLimit) {
      return await fetchReq(configProperty.data, options, body);
    } else {
      body += "&sparqlet=" + encodeURIComponent(configProperty.data) + "&limit=" + idLimit;
      return await fetchReq(sparqlSplitter, options, body);
    }
  }
  
  const getAllAttributeData = async (togoKey) => {
    let attributeData = {};
    let start = Date.now();
    // 並列
    for (let configSubject of togositeConfigJson) {
      for (let configProperty of configSubject.properties) {
        if (queryPropertyIds.includes(configProperty.propertyId)) { // クエリに Hit したら
  	     // console.log(configProperty.propertyId); // debug
          console.log(Date.now() - start);
         // attributeData[configProperty.propertyId] = await getAttributeData(togoKey, configProperty);
          attributeData[configProperty.propertyId] = getAttributeData(togoKey, configProperty).then(d => d);
       }
      }
    }
    return Promise.all(attributeData).then(d => {console.log(d); return d;});
  //  return attributeData;
  } 
  
  // label 取得
  const labelApi = "https://integbio.jp/togosite/sparqlist/api/togokey_label";
  let togoIdToLabel = {};
  if (togoKey != "togovar") {
    togoIdToLabel = await fetchReq(labelApi, options, "togoKey=" + togoKey + "&queryIds=" + queryIds);
  }
  
  const queryProperties = JSON.parse(properties);
  const queryPropertyIds = queryProperties.map(d => d.propertyId);
  const togoIdArray = JSON.parse(queryIds);
  let tableData = {};
  for (let togoId of togoIdArray) {
    tableData[togoId] = [];
  }
  
  /* let attributeData = {};
    let start = Date.now();
    // 並列
    for (let configSubject of togositeConfigJson) {
      for (let configProperty of configSubject.properties) {
        if (queryPropertyIds.includes(configProperty.propertyId)) { // クエリに Hit したら
  	     // console.log(configProperty.propertyId); // debug
          console.log(Date.now() - start);
         // attributeData[configProperty.propertyId] = await getAttributeData(togoKey, configProperty);
          attributeData[configProperty.propertyId] = getAttributeData(togoKey, configProperty).then(d => d);
       }
      }
    }
   return Promise.all(attributeData).then(attributeData => { */
    
  //let attributeData = await getAllAttributeData(togoKey);
  return await getAllAttributeData(togoKey);
 // Promise.all(attributeData).then(attributeData => {
  //  attributeData = getAllAttributeData(togoKey);
 /*   console.log("hoge");
    console.log(attributeData);
    for (let configSubject of togositeConfigJson) {
      for (let configProperty of configSubject.properties) {
        if (queryPropertyIds.includes(configProperty.propertyId)) { // クエリに Hit したら
          // mapping to 'togoKey' ID
          let primaryId2attribute = {};
          for (let d of attributeData[configProperty.propertyId]) {
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
    return togoIdArray.map(togoId=>{
      let obj = { id: togoId };
      if (togoIdToLabel[togoId]) obj.label = togoIdToLabel[togoId];
      obj.properties = tableData[togoId];
      return obj;
    })
//  }) */
}
```
