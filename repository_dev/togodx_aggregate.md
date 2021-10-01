# TogoDX aggregate SPARQList (Parameter, JSON 変更) 21.10.01

- 元 togokey_filter
- 入れ子 SPARQList の parameters もそのうち修正する
　- categoryIds -> nodes
  - togokey -> togokey
  - inputIds -> queries
 
## Parameters

* `togokey`
  * default: hgnc
* `filters`
  * default: [{"attribute": "gene_high_level_expression_refex", "nodes": ["v32_40", "v25_40"]}, {"attribute": "protein_cellular_component_uniprot","nodes": ["GO_0005886"]}, {"attribute": "structure_data_existence_uniprot", "nodes": ["1"]}, {"attribute": "interaction_chembl_assay_existence_uniprot", "nodes": ["1"]}]
* `queries` Uploaded user IDs
  * example: ["1193","13940","13557","15586","16605","4942","5344","6148", "6265","6344","6677","6735","10593","10718","10876"]
  
## `primaryIds`
```javascript
async ({togokey, filters, queries})=>{
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
  
  const togoDxAttributes = "https://raw.githubusercontent.com/togodx/togodx-config-human/develop/config/attributes.json";
  const sparqlSplitter = "togoid_sparqlist_splitter";  // nested SPARQLet relative path
  const togoidApi = "togoid_route_sparql";  // nested SPARQLet relative path
  const attributesJson = await fetchReq(togoDxAttributes, {method: "get"});
  const queryProperties = JSON.parse(filters);
  let idLimit = 2000; // split 判定
  if (togokey == "pubchem_compound") idLimit = 500; // restrict POST respons size
  const start = Date.now(); // debug

  // not filter (togoKey = hgnc, uniprot, pdb, mondo)
  const togoidNotFilter = "togokey_not_filter"; // nested SPARQLet relative path
  if (queryProperties.length == 0 && (togokey == "hgnc" || togokey == "uniprot" || togokey == "pdb" || togokey == "mondo")) {
    if (queries && JSON.parse(queries)[0]) return JSON.parse(queries);
    return fetchReq(togoidNotFilter, options, "togoKey=" + togokey);
  }

  const getIdPair = async (query) => {
    const attribute = query.attribute;
    const config = attributesJson.attributes[attribute];
     //const t1 = Date.now() - start; // debug
    
    // get 'primatyKey' ID list by category filtering
    let queryCategoryIds = query.nodes.join(",");

    config.api = config.api.split(/\//).slice(-1)[0];  // nested SPARQLet relative path
    let primaryIds = await fetchReq(config.api, options, "mode=idList&categoryIds=" + queryCategoryIds);
    //const t2 = Date.now() - start; // debug
    
    // get 'primalyKey' ID - togoKey' ID list via togoID API
    let idPair = [];
    if (togoKey != config.dataset) {
      let body = "source=" + config.dataset + "&target=" + togoKey + "&ids=" +  encodeURIComponent(primaryIds.join(","));
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
    //console.log(configProperty.attribute + ": start " + t1 + ",mid " + t2 + ",fin " + t3);
    
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
  if (queries && JSON.parse(queries)[0]) {
    togoId = {};
    for (let id of JSON.parse(queries)) {
      togoId[id] = true;
    }
  }
  
  let idPairAll = await getAllIdPair();
  // console.log(idPairAll);

  for (let i = 0; i < queryProperties.length; i++) { 
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