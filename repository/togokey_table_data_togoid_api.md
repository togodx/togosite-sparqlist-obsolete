# togokey table data (aggregate SPARQList) テーブルデータ取得（絞り込み CategoryIds あり、絞り込み無し）(TogoID API 版)

## Parameters

* `togoKey`
  * default: pubchem_compound
* `properties`
  * default: [{"propertyId":"homologene_category","categoryIds":["8"]}]
* `queryIds` togoKey 10個程度ずつ
  * default: ["129","176","177","187","222","223","271","280","312","649","681","706","712","774","784","888","923","936","943","962","977","1023","1038","1061","1135","1174","2662","3469","4973","5202","5884","5886","5994","5997","6030","6031","6047","6076","6083","6131","6133","6176","8955","9903","10214","14985","15625","15993","18058","27099","27284","27854","29936","32051","34756","35717","60961","64689","64968","65091","77213","83814","87642","91474","104815","104994","114926","122328","145068","159296","159325","160419","164533","171548","222528","439155","439156","439260","441007","444493","444795","444899","445354","445675","446013","447920","448265","451714","504166","638015","643975","643976","1549073","2733535","3434975","5257127","5280360","5280363","5280453","5280490"]

#[{"propertyId": "refex_specific_high_expression", "categoryIds": ["v32_40", "v25_40"]}, {"propertyId": "uniprot_keywords_cellular_component","categoryIds": ["472"]}, {"propertyId": "uniprot_pdb_existence", "categoryIds": ["1"]}, {"propertyId": "uniprot_chembl_assay_existence", "categoryIds": ["1"]},{"propertyId": "refex_specific_low_expression"}, {"propertyId": "uniprot_phospho_site"}, {"propertyId": "uniprot_keywords_biological_process"}]

#["4942","5344","6148", "6265","6344","6677","6735","10593","10718","10876"]

## `primaryIds`
```javascript
async ({togoKey, properties, queryIds})=>{
  let fetchReq = async (url, options, body) => {
    console.log(body); // debug
    console.log(url);  // debug
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
  
//  let togositeConfig = "https://raw.githubusercontent.com/dbcls/togosite/develop/config/togosite.config.json";
  let togositeConfig = "https://raw.githubusercontent.com/dbcls/togosite/develop/config/togosite-human/properties.json";
  let togoidApi = "https://integbio.jp/togosite/sparqlist/api/togoid_route_api"; // TogoID API 版
  let togositeConfigJson = await fetchReq(togositeConfig, {method: "get"});
 
  // label 取得
  let labelApi = "https://integbio.jp/togosite/sparqlist/api/togokey_label";
  let togoIdToLabel = {};
  if (togoKey != "togovar") {
    togoIdToLabel = await fetchReq(labelApi, options, "togoKey=" + togoKey + "&queryIds=" + queryIds);
  }
  
  let queryProperties = JSON.parse(properties);
  let queryPropertyIds = queryProperties.map(d => d.propertyId);
  let togoIdArray = JSON.parse(queryIds);
  let tableData = {};
  for (let togoId of togoIdArray) {
    tableData[togoId] = [];
  }
  // properties.json を順番に
  for (let configSubject of togositeConfigJson) {
    for (let configProperty of configSubject.properties) {
      if (queryPropertyIds.includes(configProperty.propertyId)) { // クエリに Hit したら
        console.log(configProperty.propertyId); // debug
        
        // get 'togoKey' ID - 'primalyKey' ID list via TogoID API
        let idPair = [];
        if (togoKey != configProperty.primaryKey) {
          let body = "source=" + togoKey + "&target=" + configProperty.primaryKey + "&ids=" + encodeURIComponent(togoIdArray.join(","));
          idPair = await fetchReq(togoidApi, options, body);
        } else {
          idPair = togoIdArray.map(d => {return {source_id: d, target_id: d} });
        }
        let togo2primary = {};
        for (let d of idPair) {
          if (!togo2primary[d.source_id]) togo2primary[d.source_id] = [];
          togo2primary[d.source_id].push(d.target_id);
        }
        
        for (let d of idPair) {
          if (d.source_id == "649") console.log(d); 
        }

        // get attributes of 'primaryKey' Ids
        let primaryIds = Array.from(new Set(idPair.map(d=>d.target))).join(" ");
        let categoryIdsParam = "";
        for (let queryProperty of queryProperties) {
          if (queryProperty.propertyId == configProperty.propertyId) {
            if (queryProperty.categoryIds) categoryIdsParam = "&categoryIds=" + queryProperty.categoryIds.join(",");
            break;
          }
        }
        let objectList = await fetchReq(configProperty.data, options, "mode=objectList&queryIds=" + encodeURIComponent(primaryIds) + categoryIdsParam);

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
    let obj = {
      id: togoId,
      properties: tableData[togoId]
    }
    if (togoIdToLabel[togoId]) obj.label = togoIdToLabel[togoId];
    return obj;
  });
}
```

