# TogoDX dataframe SPARQList (Map attributesの内訳指定を親階層での選択版) 21.09.30

- 元 togokey_table_data

## Parameters

* `togoKey`
  * default: hgnc
* `properties`
  * default: {"filter": [{"propertyId": "gene_high_level_expression_refex", "categoryIds": ["v32_40", "v25_40"]}, {"propertyId": "protein_cellular_component_uniprot","categoryIds": ["GO_0005886"]}, {"propertyId": "structure_data_existence_uniprot", "categoryIds": ["1"]}, {"propertyId": "interaction_chembl_assay_existence_uniprot", "categoryIds": ["1"]}], "mapping": [{"propertyId": "gene_low_level_expression_refex"}, {"propertyId": "protein_number_of_phosphorylation_sites_uniprot"}, {"propertyId": "protein_biological_process_uniprot", "categoryIds": ["GO_0009987"]}]}
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
  
  const togositeConfig = "https://raw.githubusercontent.com/togodx/togodx-config-human/develop/config/properties.json";
  const sparqlSplitter = "sparqlist_splitter"; // nested SPARQLet relative path
  const togoidApi = "togoid_route_sparql";  // nested SPARQLet relative path
  const labelApi = "togokey_label";  // nested SPARQLet relative path
  const togositeConfigJson = await fetchReq(togositeConfig, {method: "get"});
  const idLimit = 2000; // split 判定
  const queryPropertiesPre = JSON.parse(properties);
  const togoIdArray = JSON.parse(queryIds);
  const start = Date.now(); // debug

  let queryProperties = [];
  if (queryPropertiesPre.filter) queryProperties = queryPropertiesPre.filter;
  if (queryPropertiesPre.mapping) queryProperties = queryProperties.concat(queryPropertiesPre.mapping.map(d => { d.mapping = true; return d; }));
  
  let propertyId2config = {}
  for (let configSubject of togositeConfigJson) {
    for (let configProperty of configSubject.properties) {
      propertyId2config[configProperty.propertyId] = configProperty;
    }
  }
  
  const getAttributeData = async (query) => {
    const propertyId = query.propertyId;
    const config = propertyId2config[propertyId];
    config.data = config.data.split(/\//).slice(-1)[0]; // nested SPARQLet relative path
    //const t1 = Date.now() - start; // debug
    
    // get 'togoKey' ID - 'primalyKey' ID list via TogoID API
    let idPair = [];
    if (togoKey != config.primaryKey) idPair = await fetchReq(togoidApi, options, "source=" + togoKey + "&target=" + config.primaryKey + "&ids=" + encodeURIComponent(togoIdArray.join(" ")));
    else idPair = togoIdArray.map(d => {return {source_id: d, target_id: d} });
    let togo2primary = {};
    for (let d of idPair) {
      if (!togo2primary[d.source_id]) togo2primary[d.source_id] = [];
      togo2primary[d.source_id].push(d.target_id);
    }
    //const t2 = Date.now() - start; // debug
    
    // get attributes of 'primaryKey' Ids
    let primaryIds = Array.from(new Set(idPair.map(d=>d.target_id))).join(",");
    let categoryIdsParam = "";
    if (query.categoryIds) categoryIdsParam = "&categoryIds=" + query.categoryIds.join(",");

    let body = "mode=objectList&queryIds=" + encodeURIComponent(primaryIds) + categoryIdsParam;
    let objectList = [];
    if (primaryIds.length <= idLimit) {
      objectList = await fetchReq(config.data, options, body);
    } else {
      body += "&sparqlet=" + encodeURIComponent(config.data) + "&limit=" + idLimit;
      objectList =  await fetchReq(sparqlSplitter, options, body);
    }  
    //const t3 = Date.now() - start; // debug
    //console.log(propertyId + ": start " + t1 + ",mid " + t2 + ",fin " + t3);
    
    // map attribute for lower-level category 下層カテゴリへのマッピングのための処理
    if (query.mapping && query.categoryIds) {
      let body = "categoryIds=" + query.categoryIds.join(",");
      let lowClass = await fetchReq(config.data, options, body);
      //console.log(lowClass.map(d => d.categoryId).join(","));
      body = "mode=objectList&queryIds=" + encodeURIComponent(primaryIds) + "&categoryIds=" + lowClass.map(d => d.categoryId).join(",");
      let objectList_2 = [];
      if (primaryIds.length <= idLimit) {
        objectList_2 = await fetchReq(config.data, options, body);
      } else {
        body += "&sparqlet=" + encodeURIComponent(config.data) + "&limit=" + idLimit;
        objectList_2 =  await fetchReq(sparqlSplitter, options, body);
      }
      let withLowCategory = {}
      objectList_2.forEach(d => {
        withLowCategory[d.id] = true;
      })
      objectList = objectList_2.concat(objectList.filter(d => !withLowCategory[d.id]))
    }
    
    return {"pair": togo2primary, "list": objectList};
  }
  
  const getAllAttributeData = async () => {
    let attributeData = {};
    for (let i = 0; i < queryProperties.length; i++) {
      attributeData[i] = getAttributeData(queryProperties[i]).then(d => d);
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

  for (let i = 0; i < queryProperties.length; i++) {
    const propertyId = queryProperties[i].propertyId;     
    const config = propertyId2config[propertyId]; 
    
    let attributeData = await attributeDataAll[i];
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
      tableData[togoId].push({
        propertyId: propertyId,
        propertyLabel: config.label,
        propertyKey: config.primaryKey,
        attributes: attributeList
      })
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