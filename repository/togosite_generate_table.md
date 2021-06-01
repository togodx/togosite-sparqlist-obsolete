# togosite generate table

togokey table data (aggregate SPARQList) テーブルデータ取得（絞り込み CategoryIds あり、絞り込み無し）

## Parameters

* `togoKey`
  * default: hgnc
* `properties`
  * default: [{"propertyId": "refex_specific_high_expression", "categoryIds": ["v32_40", "v25_40"]}, {"propertyId": "uniprot_keywords_cellular_component","categoryIds": ["472"]}, {"propertyId": "uniprot_pdb_existence", "categoryIds": ["1"]}, {"propertyId": "uniprot_chembl_assay_existence", "categoryIds": ["1"]},{"propertyId": "refex_specific_low_expression"}, {"propertyId": "uniprot_phospho_site"}, {"propertyId": "uniprot_keywords_biological_process"}]
* `queryIds` togoKey 100個程度ずつ
  * default: ["4942","5344","6148", "6265","6344","6677","6735","10593","10718","10876"]

## `main`

```javascript
async ({togoKey, properties, queryIds})=>{
  const propertiesConfigJson = "https://raw.githubusercontent.com/dbcls/togosite/develop/config/togosite-human/properties.json";
  const togoidApi            = "https://integbio.jp/togosite/sparqlist/api/togoid_route_sparql";
  const propertyIdList       = JSON.parse(properties).map(x => x.propertyId);

  const fetchReq = async (url, options, body) => {
    console.log(body); // debug
    console.log(url);  // debug
    if (body) options.body = body;
    return await fetch(url, options).then(res=>res.json());
  }

  // create property key dict: e.g. {"ensembl-gene-biotype": "ensembl_gene"}
  const createPropertyKeyDict = async (properties) => {
    const propertyKeyDict = {};
    for (const subject in propertiesConfigJson) {
      for (const property in subject.properties) {
        if (propertyIdList.includes(property.propertyId)) {
          propertyKeyDict[property.propertyId] = property.primaryKey;
        }
      }
    }
    return propertyKeyDict;
  }

  // Convert id function returns [{"source_id": "4942", "target_id": "ENSG00000196735"}]
  const convertId = async (fromId, toId, ids) => {
    if (fromId == toId) {
      ids.map(d => { return { source_id: d, target_id: d } });
    } else {
      await fetchReq(
        togoidApi,
        options,
        "source=" + fromId + "&target=" + toId + "&ids=" + encodeURIComponent(ids.join(" "))
      )
    }
  }

  // create id dict: {"4942" => {"ensembl_gene": ["ENSG00000196735"]} }
  const createIdDict = async (togoKey, properties, queryIds) => {
    const dict = await createPropertyKeyDict(properties);
    for (const property in properties) {
      const propertyId = property.propertyId;
      const propertyKey = dict[propertyId];
      convertId(togoKey, propertyKey, queryIds)
    }
  }

  return createIdDict(togoKey, properties, queryIds);
}
```

