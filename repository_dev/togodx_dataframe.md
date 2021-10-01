# TogoDX dataframe SPARQList (Parameter, JSON 変更) 21.10.01

- 元 togokey_table_data
- 入れ子 SPARQList の parameters もそのうち修正する
  - categoryIds -> nodes
  - togokey -> togokey
  - queryIds -> queries

## Parameters

* `togokey`
  * default: hgnc
* `filters`
  * default: [{"attribute": "gene_high_level_expression_refex", "nodes": ["v32_40", "v25_40"]}, {"attribute": "protein_cellular_component_uniprot","nodes": ["GO_0005886"]}, {"attribute": "structure_data_existence_uniprot", "nodes": ["1"]}, {"attribute": "interaction_chembl_assay_existence_uniprot", "nodes": ["1"]}]
* `annotations`
  * default: [{"attribute": "gene_low_level_expression_refex"}, {"attribute": "protein_number_of_phosphorylation_sites_uniprot"}, {"attribute": "protein_biological_process_uniprot", "node": "GO_0009987"}]
* `queries` togokey 100個程度ずつ
  * default: ["4942","5344","6148", "6265","6344","6677","6735","10593","10718","10876"]

## `primaryIds`
```javascript
async ({togokey, filters, annotations, queries})=>{
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
  
  const togoDxAttributes = "https://raw.githubusercontent.com/togodx/togodx-config-human/develop/config/attributes.json";
  const sparqlSplitter = "sparqlist_splitter"; // nested SPARQLet relative path
  const togoidApi = "togoid_route_sparql";  // nested SPARQLet relative path
  const labelApi = "togokey_label";  // nested SPARQLet relative path
  const attributesJson = await fetchReq(togoDxAttributes, {method: "get"});
  const idLimit = 2000; // split 判定
  const queryFilters = JSON.parse(filters);
  const queryAnnotations= JSON.parse(annotations);
  const togoIdArray = JSON.parse(queries);
  const start = Date.now(); // debug

  let queryProperties = [];
  if (queryFilters) queryProperties = queryFilters;
  if (queryAnnotations) queryProperties = queryProperties.concat(queryAnnotations.map(d => { d.annotation = true; return d; }));
  
  const getAttributeData = async (query) => {
    const attribute = query.attribute;
    const config = attributesJson.attributes[attribute];
    config.api = config.api.split(/\//).slice(-1)[0]; // nested SPARQLet relative path
    //const t1 = Date.now() - start; // debug
    
    // get 'togokey' ID - 'primalyKey' ID list via TogoID API
    let idPair = [];
    if (togokey != config.dataset) idPair = await fetchReq(togoidApi, options, "source=" + togokey + "&target=" + config.dataset + "&ids=" + encodeURIComponent(togoIdArray.join(" ")));
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
    if (query.nodes) categoryIdsParam = "&categoryIds=" + query.nodes.join(",");  // #### 入れ子 SPARQList. 要パラメータ名の整理
    if (query.node) categoryIdsParam = "&categoryIds=" + query.node;  // #### 入れ子 SPARQList. 要パラメータ名の整理
    
    let body = "mode=objectList&queryIds=" + encodeURIComponent(primaryIds) + categoryIdsParam;  // #### 入れ子 SPARQList. 要パラメータ名の整理
    let objectList = [];
    if (primaryIds.length <= idLimit) {
      objectList = await fetchReq(config.api, options, body);
    } else {
      body += "&sparqlet=" + encodeURIComponent(config.api) + "&limit=" + idLimit;
      objectList =  await fetchReq(sparqlSplitter, options, body);
    }  
    //const t3 = Date.now() - start; // debug
    //console.log(attribute + ": start " + t1 + ",mid " + t2 + ",fin " + t3);
    
    // map attribute for lower-level category 下層カテゴリへのマッピングのための処理
    if (query.annotation && query.node) {
      let body = "categoryIds=" + query.node;    // #### 入れ子 SPARQList. 要パラメータ名の整理
      let lowClass = await fetchReq(config.api, options, body);
      //console.log(lowClass.map(d => d.categoryId).join(","));
      body = "mode=objectList&queryIds=" + encodeURIComponent(primaryIds) + "&categoryIds=" + lowClass.map(d => d.categoryId).join(",");  // #### 入れ子 SPARQList. 要パラメータ名の整理
      let objectList_2 = [];
      if (primaryIds.length <= idLimit) {
        objectList_2 = await fetchReq(config.api, options, body);
      } else {
        body += "&sparqlet=" + encodeURIComponent(config.api) + "&limit=" + idLimit;
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
  let togoIdToLabelFetch = fetchReq(labelApi, options, "togoKey=" + togokey + "&queryIds=" + queries);  // #### 入れ子 SPARQList. 要パラメータ名の整理

  let attributeDataAll = await getAllAttributeData(); // promise
  // console.log(attributeDataAll);

  for (let i = 0; i < queryProperties.length; i++) {
    const attribute = queryProperties[i].attribute;     
    const config = attributesJson.attributes[attribute];
    
    let attributeData = await attributeDataAll[i];
    let togo2primary = attributeData.pair;
	let objectList = attributeData.list;
          
    // mapping to 'togokey' ID
    let primaryId2attribute = {};
    for (let d of objectList) {
      if (!primaryId2attribute[d.id]) primaryId2attribute[d.id] = [];
      primaryId2attribute[d.id].push(d);
    }
    for (let togoId of Object.keys(togo2primary)) {
      let leafList = [];
      for (let primaryId of togo2primary[togoId]) {
        if (primaryId2attribute[primaryId]) leafList = leafList.concat(primaryId2attribute[primaryId]);
      }
      tableData[togoId].push({
        propertyId: attribute,
        propertyLabel: config.label,
        propertyKey: config.dataset,
        attributes: leafList
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