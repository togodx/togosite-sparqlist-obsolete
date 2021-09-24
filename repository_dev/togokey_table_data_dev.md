# togokey table data (aggregate SPARQList)(非同期版)テーブルデータ取得（絞り込み CategoryIds あり、絞り込み無し）

## Parameters

* `togoKey`
  * default: ensembl_gene
* `properties`
  * default:  [{"propertyId":"gene_evolutionary_conservation_homologene","categoryIds":["5"]}]
  *example: [{"propertyId":"gene_evolutionary_conservation_homologene","categoryIds":["5"]},{"propertyId":"protein_biological_process_uniprot"},{"propertyId":"protein_biological_process_uniprot","categoryIds":["GO_0050789","GO_0065008","GO_0065009"]}]
* `queryIds` togoKey 100個程度ずつ
  * default: ["ENSG00000169221"]
  *example: ["ENSG00000169221","ENSG00000253159","ENSG00000135631","ENSG00000276040","ENSG00000137098","ENSG00000124657","ENSG00000092330","ENSG00000117215","ENSG00000185385","ENSG00000196184"]

## `primaryIds`
```javascrip
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
  const queryProperties = JSON.parse(properties);
  const queryPropertyIds = queryProperties.map(d => d.propertyId);
  const togoIdArray = JSON.parse(queryIds);
  const start = Date.now(); // debug
  
  const getAttributeData = async (configProperty) => {
    configProperty.data = configProperty.data.split(/\//).slice(-1)[0]; // nested SPARQLet relative path
    //const t1 = Date.now() - start; // debug
    
    // get 'togoKey' ID - 'primalyKey' ID list via TogoID API
    let idPair = [];
    if (togoKey != configProperty.primaryKey) idPair = await fetchReq(togoidApi, options, "source=" + togoKey + "&target=" + configProperty.primaryKey + "&ids=" + encodeURIComponent(togoIdArray.join(" ")));
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
    for (let queryProperty of queryProperties) {
      if (queryProperty.propertyId == configProperty.propertyId) {
        if (queryProperty.categoryIds) categoryIdsParam = "&categoryIds=" + queryProperty.categoryIds.join(",");
        break;
      }
    }
    let body = "mode=objectList&queryIds=" + encodeURIComponent(primaryIds) + categoryIdsParam;
    let objectList = [];
    if (primaryIds.length <= idLimit) {
      objectList = await fetchReq(configProperty.data, options, body);
    } else {
      body += "&sparqlet=" + encodeURIComponent(configProperty.data) + "&limit=" + idLimit;
      objectList =  await fetchReq(sparqlSplitter, options, body);
    }  
    //const t3 = Date.now() - start; // debug
    //console.log(configProperty.propertyId + ": start " + t1 + ",mid " + t2 + ",fin " + t3);
    
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