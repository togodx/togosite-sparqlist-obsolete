# Proteins lowly expressed in tissues (HPA)（小野・池田・千葉）

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters
* `categoryIds` (type: CALOHA)
  * example: TS-0892,TS-0013,TS-0440
* `queryIds` (type: ensembl_gene)
  * example: ENSG00000000005,ENSG00000002587,ENSG00000003989
* `mode`
  * example: idList, objectList

## `httpreq`

```javascript
async ({queryIds, categoryIds, mode})=>{
  let url = "protein_expression_level_in_tissues_hpa"; // parent SPARQLet relative path
  let options = {
    method: 'POST',
    body	: "level=Low&categoryIds=" + categoryIds + "&queryIds=" + encodeURIComponent(queryIds) + "&mode=" + mode,
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  return await fetch(url, options).then(res=>res.json());
}
```
