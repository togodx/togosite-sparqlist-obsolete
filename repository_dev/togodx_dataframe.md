# TogoDX dataframe SPARQList (JSON 変更) 22.01.18

- 入れ子 SPARQList の parameters もそのうち修正する
  - categoryIds -> nodes
  - togokey -> dataset
  - queryIds -> queries

## Parameters

* `dataset`
  * default: ensembl_gene
* `filters`
  * default: [{"attribute": "gene_evolutionary_conservation_homologene","nodes": ["6"]}]
  * example: [{"attribute": "gene_high_level_expression_refex", "nodes": ["v32_40", "v25_40"]}, {"attribute": "protein_cellular_component_uniprot","nodes": ["GO_0005886"]}, {"attribute": "structure_data_existence_uniprot", "nodes": ["1"]}, {"attribute": "interaction_chembl_assay_existence_uniprot", "nodes": ["1"]}]
* `annotations`
  * default: [{"attribute": "protein_biological_process_uniprot"},{"attribute": "protein_biological_process_uniprot", "node": "GO_0044237"}]
  * example: [{"attribute": "gene_low_level_expression_refex"}, {"attribute": "protein_number_of_phosphorylation_sites_uniprot"}, {"attribute": "protein_biological_process_uniprot", "node": "GO_0009987"}]
* `queries` 100個程度ずつ
  * default: ["ENSG00000173230"]
  * example: ["4942","5344","6148", "6265","6344","6677","6735","10593","10718","10876"]

## `primaryIds`
```javascript
async ({dataset, filters, annotations, queries})=>{
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
    
    // get 'dataset' ID - 'primalyKey' ID list via TogoID API
    let idPair = [];
    if (dataset != config.dataset) idPair = await fetchReq(togoidApi, options, "source=" + dataset + "&target=" + config.dataset + "&ids=" + encodeURIComponent(togoIdArray.join(" ")));
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
  let togoIdToLabelFetch = fetchReq(labelApi, options, "togoKey=" + dataset + "&queryIds=" + queries);  // #### 入れ子 SPARQList. 要パラメータ名の整理

  let attributeDataAll = await getAllAttributeData(); // promise
  // console.log(attributeDataAll);

  for (let i = 0; i < queryProperties.length; i++) {
    const attribute = queryProperties[i].attribute;     
    const config = attributesJson.attributes[attribute];
    
    let attributeData = await attributeDataAll[i];
    let togo2primary = attributeData.pair;
	let objectList = attributeData.list;
          
    // mapping to 'dataset' ID
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
        id: attribute,
        //propertyLabel: config.label,
        items: leafList.map(d => {
          return {
            datadset: config.dataset,
            entry: d.id,
            node: d.attribute.categoryId,
            label: d.attribute.label
          }
        })  
      })
    }
  }
  
  // object to list
  let togoIdToLabel = await togoIdToLabelFetch;
  return togoIdArray.map(togoId=>{
    let obj = { index: {dataset: dataset, entry: togoId}};
    if (togoIdToLabel[togoId]) obj.index.label = togoIdToLabel[togoId];
    obj.attributes = tableData[togoId];
    return obj;
  })
}
```