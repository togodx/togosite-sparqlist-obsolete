# variant clinical significance (三橋）

## Description

- Data sources
    -  [TogoVar](https://togovar.biosciencedbc.jp/?) (limited to variants with frequency data in Japanese populations)
- Query
    - Input
        - TogoVar id
    - Output
        -   [Clinical significance of ClinVar](https://www.ncbi.nlm.nih.gov/clinvar/docs/clinsig/)

## Parameters

* `categoryIds` (type:Clinical significance from ClinVar)
  * example: uncertain_significance,likely_benign,benign,pathogenic,likely_pathogenic,conflicting_interpretations_of_pathogenicity,not_provided,benign_or_likely_benign,pathogenic_or_likely_pathogenic,other,drug_response,risk_factor,association,affects,protective
* `queryIds` (type:TogoVar)
  * example: tgv48208871,tgv48208872,tgv48208877,tgv48208884,tgv48208888,tgv48208896,tgv48208897,tgv48208898,tgv48208900,tgv48208904,tgv48208906,tgv48208908,tgv48208938,tgv48208939,tgv48208943,tgv48208945,tgv48208947,tgv48208962,tgv48208965,tgv48208967,tgv48208968,tgv48208970,tgv48208971,tgv48208973,tgv48208977
* `mode` 必須パラメータ。内訳の代わりに該当する ID のリストを返す（デフォルトはオフ）idList: リストだけ、objectList: Attributeの入ったリスト（Attribute は下階層ではなく、categoryid で指定したカテゴリ）
  * example: idList, objectList

## testURL
- [default](https://integbio.jp/togosite/sparqlist/api/variant_clinical_significance?categoryIds=&queryIds=&mode=)
- [queryId+categoryId](https://integbio.jp/togosite/sparqlist/api/variant_clinical_significance?categoryIds=uncertain_significance%2Clikely_benign%2Cpathogenic&queryIds=tgv48208871%2Ctgv48208872%2Ctgv48208877&mode=)
- [queryId+categoryId+idList](https://integbio.jp/togosite/sparqlist/api/variant_clinical_significance?categoryIds=uncertain_significance%2Clikely_benign%2Cbenign%2Cpathogenic&queryIds=tgv48208871%2Ctgv48208872%2Ctgv48208877&mode=idList)
- [queyId+categoryId+objectList](https://integbio.jp/togosite/sparqlist/api/variant_clinical_significance?categoryIds=uncertain_significance%2Clikely_benign%2Cbenign%2Cpathogenic&queryIds=tgv48208871%2Ctgv48208872%2Ctgv48208877&mode=objectList)

## `queryArray`
- Query TogoVarIDを配列に
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## `categoryArray`
- Clinical siginificanceのIDをlabel(ClinVarの表記と同じ)に変換して配列に代入する。
```javascript
({categoryIds}) => {
  
  var id2label = {};
  id2label["uncertain_significance"] = "Uncertain significance";
  id2label["likely_benign"] = "Likely beneign"
  id2label["pathogenic"] = "Pathogenic"
  id2label["likely_pathogenic"] = "Likely pathogenic"
  id2label["conflicting_interpretations_of_pathogenicity"] = "Conflicting interpretations of pathogenicity"
  id2label["benign_or_likely_benign"] = "Benign/Likely benign"
  id2label["pathogenic_or_likely_pathogenic"] = "Pathogenic/Likely pathogenic"
  id2label["other"] = "other"
  id2label["drug_response"] = "drug response"
  id2label["risk_factor"] = "risk factor"
  id2label["association"] = "association"
  id2label["affects"] = "affects"
  id2label["protective"] = "protective"
  
  categoryIds = categoryIds.replace(/,/g," ")
  if (categoryIds.match(/[^\s]/)) return categoryIds.split(/\s+/).map( categoryId => id2label[categoryId]　);
  return false;
}
```

## Endpoint

https://integbio.jp/togosite/sparql

## `data`
```sparql

PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX tgvo: <http://togovar.biosciencedbc.jp/vocabulary/>
PREFIX cvo:  <http://purl.jp/bio/10/clinvar/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX dct: <http://purl.org/dc/terms/>

{{#if mode}}
SELECT DISTINCT ?tgv_id ?category ?category AS ?label
{{else}}
SELECT ?category ?category AS ?label (COUNT (DISTINCT ?tgv_id) AS ?count) 
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/variation>
FROM <http://rdf.integbio.jp/dataset/togosite/variation/annotation/clinvar>
FROM <http://rdf.integbio.jp/dataset/togosite/clinvar>
WHERE {  
{{#if queryArray}}
  VALUES ?tgv_id { {{#each queryArray}} "{{this}}" {{/each}} }
{{/if}}
{{#if categoryArray}}
  VALUES ?category { {{#each categoryArray}} "{{this}}" {{/each}} }    
{{/if}}
  ?togovar dct:identifier ?tgv_id.
  ?togovar tgvo:condition/rdfs:seeAlso/cvo:interpreted_record/cvo:rcv_list/cvo:rcv_accession/cvo:interpretation ?category.  
}
{{#unless mode}}  
  ORDER BY DESC(?count)
{{/unless}}
```

## `return`
- 整形
```javascript
({mode, data})=>{
  const idVarName = "tgv_id";
  const idPrfix = "";
  const categoryPrefix = "";
  if (mode == "objectList") return data.results.bindings.map(d=>{
    return {
      id: d[idVarName].value.replace(idPrfix, ""), 
      attribute: {
        categoryId: d.category.value.toLowerCase().replace("/", "_or_").replace(/,?\s+/g, "_"),
        uri: d.category.value.toLowerCase().replace("/", "_or_").replace(/,?\s+/g, "_"),
        label : d.label.value.charAt(0).toUpperCase() + d.label.value.slice(1)   // 先頭の１文字だけを大文字にする。
      }
    }
  });
  if (mode == "idList") return Array.from(new Set(data.results.bindings.map(d=>d[idVarName].value.replace(idPrfix, "")))); // unique

  return data.results.bindings.map(d=>{ 
    return {
      categoryId: d.category.value.toLowerCase().replace("/", "_or_").replace(/,?\s+/g, "_"),
      label: d.label.value.charAt(0).toUpperCase() + d.label.value.slice(1),   // 先頭の１文字だけを大文字にする。
      count: Number(d.count.value),
      hasChild: false      // ClinVarは無階層のため常にfalse
    };
  });	
}
```