## togovar_paper_significance - Togosite用途外のSPARQLet(三橋）

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
  * default: pathogenic,pathogenic_association,pathogenic_drug_response_protective_risk factor,pathogeniclikely_pathogenic,pathogenic_or_likely_pathogenic,drug_response,risk_factor,association,affects,protective
  * example: uncertain_significance,likely_benign,benign,pathogenic,likely_pathogenic,conflicting_interpretations_of_pathogenicity,not_provided,benign_or_likely_benign,pathogenic_or_likely_pathogenic,other,drug_response,risk_factor,association,affects,protective
* `queryIds` (type:TogoVar)
  * example: tgv48208871,tgv48208872,tgv48208877,tgv48208884,tgv48208888,tgv48208896,tgv48208897,tgv48208898,tgv48208900,tgv48208904,tgv48208906,tgv48208908,tgv48208938,tgv48208939,tgv48208943,tgv48208945,tgv48208947,tgv48208962,tgv48208965,tgv48208967,tgv48208968,tgv48208970,tgv48208971,tgv48208973,tgv48208977
* `mode` 必須パラメータ。内訳の代わりに該当する ID のリストを返す（デフォルトはオフ）idList: リストだけ、objectList: Attributeの入ったリスト（Attribute は下階層ではなく、categoryid で指定したカテゴリ）
  * default: objectList
  * example: objectList

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
  - [IDとClinVar表記の対応表](https://docs.google.com/spreadsheets/d/1qEy1uyS24AwlhfmNGdXWHLZv16ebvtCTa4W5dK28lwg/edit?usp=sharing)
  - [ClinVarのClinical significance一覧を取得するSPARQL](https://is.gd/01zgpr)
```javascript
({categoryIds}) => {
  
  var id2label = {};
  id2label["affects"] = "Affects"
  id2label["association_not_found"] = "association not found"
  id2label["association"] = "association"
  id2label["benign_or_likely_benign_other"] = "Benign/Likely benign, other"
  id2label["benign_or_likely_benign_risk_factor"] = "Benign/Likely benign, risk factor"
  id2label["benign_or_likely_benign"] = "Benign/Likely benign"
  id2label["benign_other"] = "Benign, other"
  id2label["benign_risk_factor"] = "Benign, risk factor"
  id2label["benign"] = "Benign"
  id2label["confers_sensitivity"] = "confers sensitivity"
  id2label["conflicting_interpretations_of_pathogenicity_association_risk_factor"] = "Conflicting interpretations of pathogenicity, association, risk factor"
  id2label["conflicting_interpretations_of_pathogenicity_association"] = "Conflicting interpretations of pathogenicity, association"
  id2label["conflicting_interpretations_of_pathogenicity_other"] = "Conflicting interpretations of pathogenicity, other"
  id2label["conflicting_interpretations_of_pathogenicity_risk_factor"] = "Conflicting interpretations of pathogenicity, risk factor"
  id2label["conflicting_interpretations_of_pathogenicity"] = "Conflicting interpretations of pathogenicity"
  id2label["drug_response"] = "drug response"
  id2label["likely_benign_other"] = "Likely benign, other"
  id2label["likely_benign_risk_factor"] = "Likely benign, risk factor"
  id2label["likely_benign"] = "Likely benign"
  id2label["likely_pathogenic_affects"] = "Likely pathogenic, Affects"
  id2label["likely_pathogenic_drug_response"]="Likely pathogenic, drug response"
  id2label["likely_pathogenic_other"] = "Likely pathogenic, other"
  id2label["likely_pathogenic_risk_factor"] = "Likely pathogenic, risk factor"
  id2label["likely_pathogenic"] = "Likely pathogenic"
  id2label["not_provided"] = "not provided"
  id2label["other_risk_factor"] = "other, risk factor"
  id2label["other"] = "other"
  id2label["pathogenic_affects"] = "Pathogenic, Affects"
  id2label["pathogenic_association"] = "Pathogenic, association"
  id2label["pathogenic_association_protective"] = "Pathogenic, association, protective"
  id2label["pathogenic_drug_response"] = "Pathogenic, drug response"
  id2label["pathogenic_drug_response_other"] = "Pathogenic, drug response, other"
  id2label["pathogenic_drug_response_protective_risk factor"]="Pathogenic, drug response, protective, risk factor"
  id2label["pathogenic_other"]="Pathogenic, other"
  id2label["pathogenic_protective"]="Pathogenic, protective"
  id2label["pathogenic_risk_factor"] = "Pathogenic, risk factor"
  id2label["pathogenic"] = "Pathogenic"
  id2label["pathogenic_or_likely_pathogenic_association"] = "Pathogenic/Likely pathogenic, association"
  id2label["pathogenic_or_likely_pathogenic_drug_response"] = "Pathogenic/Likely pathogenic, drug response"
  id2label["pathogenic_or_likely_pathogenic_other"] = "Pathogenic/Likely pathogenic, other"
  id2label["pathogenic_or_likely_pathogenic_risk_factor"] = "Pathogenic/Likely pathogenic, risk factor"
  id2label["pathogenic_or_likely_pathogenic"] = "Pathogenic/Likely pathogenic"
  id2label["protective"] = "protective"
  id2label["protective_risk_factor"] = "protective, risk factor"
  id2label["risk_factor"] = "risk factor"
  id2label["uncertain_significance_affects"] = "Uncertain significance, Affects"
  id2label["uncertain_significance_other"] = "Uncertain significance, other"
  id2label["uncertain_significance_risk_factor"] = "Uncertain significance, risk factor"
  id2label["uncertain_significance"] = "Uncertain significance"
  
  categoryIds = categoryIds.replace(/,/g," ")
  if (categoryIds.match(/[^\s]/)) return categoryIds.split(/\s+/).map( categoryId => id2label[categoryId]　);
  return false;
}
```

## Endpoint

https://integbio.jp/togosite/sparql

## `data`
```sparql
PREFIX cvo:  <http://purl.jp/bio/10/clinvar/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX tgvo: <http://togovar.biosciencedbc.jp/vocabulary/>

{{#if mode}}
SELECT DISTINCT ?togovar ?tgv_id ?vcv_id ?vcv_clinical_significance ?rcv_id ?rcv_clinical_significance ?condition
{{else}}
SELECT ?vcv_clinical_significance (COUNT (DISTINCT ?tgv_id) AS ?count) 
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/variation>
FROM <http://rdf.integbio.jp/dataset/togosite/variation/annotation/clinvar>
FROM <http://rdf.integbio.jp/dataset/togosite/clinvar>
WHERE {  
{{#if queryArray}}
  VALUES ?tgv_id { {{#each queryArray}} "{{this}}" {{/each}} }
  ?togovar dct:identifier ?tgv_id.
{{/if}}
{{#if categoryArray}}
  VALUES ?vcv_clinical_significance { {{#each categoryArray}} "{{this}}" {{/each}} }    
{{/if}}
  ?togovar dct:identifier ?tgv_id.
  ?togovar tgvo:condition/rdfs:seeAlso ?clinvar.
  ?clinvar cvo:interpreted_record /sio:SIO_000255 ?typed_aggregated_interpretation.
  ?clinvar cvo:accession ?vcv_id.
  ?typed_aggregated_interpretation cvo:description ?vcv_clinical_significance.
  ?typed_aggregated_interpretation cvo:type "Clinical significance".  
  ?togovar tgvo:condition/rdfs:seeAlso/cvo:interpreted_record/cvo:rcv_list/cvo:rcv_accession ?rcv.
  ?rcv cvo:accession ?rcv_id.
  ?rcv cvo:interpretation ?rcv_clinical_significance. 
  ?rcv cvo:interpreted_condition_list/cvo:interpreted_condition/rdfs:label ?condition.
#  FILTER(?vcv_clinical_significance != ?rcv_clinical_significance).
}
{{#if mode}}  
#limit 50000 
{{else}}
ORDER BY DESC(?count)
{{/if}}
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
      togovar : d.togovar.value,
      tgv_id : d.tgv_id.value,
      vcv_id : d.vcv_id.value,
      vcv_clinical_significance : d.vcv_clinical_significance.value,
      rcv_id : d.rcv_id.value,
      rcv_clinical_significance : d.rcv_clinical_significance.value,
      condition : d.condition.value
    }
  });

  return data.results.bindings.map(d=>{ 
    return {
      vcv_clinical_significance : d.vcv_clinical_significance.value,
      count: Number(d.count.value)
    };
  });	
}
```
