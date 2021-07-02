# variant GWAS (Genome-wide association study) 

## Description

- Data sources
    -  [TogoVar](https://togovar.biosciencedbc.jp/?) (limited to variants with frequency data in Japanese populations)
- Query
    - Input
        - TogoVar id
    - Output
        -  [Mapped trait used in GWAS Catalog](https://)

## Parameters

* `categoryIds` (type: Mapped trait in terms of Experimental factor ontology (EFO))
  * default: EFO_0000408
  * example: EFO_0000408,EFO_0000001,IAO_0000030 
* `queryIds` (type:TogoVar)
  * example: tgv704775,tgv704941,tgv772580,tgv246970,tgv39969772,tgv40054079,tgv42043030
* `mode` 必須パラメータ。内訳の代わりに該当する ID のリストを返す（デフォルトはオフ）idList: リストだけ、objectList: Attributeの入ったリスト（Attribute は下階層ではなく、categoryid で指定したカテゴリ）
  * example: idList, objectList
* `endpoint` Endpoint
  * default: https://test100.biosciencedbc.jp/sparql

## testURL
- [default](https://integbio.jp/togosite/sparqlist/api/variant_clinical_significance?categoryIds=&queryIds=&mode=)
- [queryId+categoryId](https://integbio.jp/togosite/sparqlist/api/variant_clinical_significance?categoryIds=uncertain_significance%2Clikely_benign%2Cpathogenic&queryIds=tgv48208871%2Ctgv48208872%2Ctgv48208877&mode=)
- [queryId+categoryId+idList](https://integbio.jp/togosite/sparqlist/api/variant_clinical_significance?categoryIds=uncertain_significance%2Clikely_benign%2Cbenign%2Cpathogenic&queryIds=tgv48208871%2Ctgv48208872%2Ctgv48208877&mode=idList)
- [queyId+categoryId+objectList](https://integbio.jp/togosite/sparqlist/api/variant_clinical_significance?categoryIds=uncertain_significance%2Clikely_benign%2Cbenign%2Cpathogenic&queryIds=tgv48208871%2Ctgv48208872%2Ctgv48208877&mode=objectList)

## `queryArray`
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ").replace(/^\s+/,"").replace(/\s+$/,"");
  return (queryIds == "" ? false : queryIds.split(/\s+/).map(id=>'"' + id + '"'));
}
```

## `categoryArray`
```javascript
({categoryIds}) => {
  categoryIds = categoryIds.replace(/,/g," ").replace(/^\s+/,"").replace(/\s+$/,"");
  if(categoryIds == ""){ return false; }
  else {
    return categoryIds.split(/\s+/).map(categoryId=>{
      return categoryId.replace("EFO_","efo:EFO_").replace("MONDO_","mondo:MONDO_"); });
  }
}
```

## Endpoint

https://test100.biosciencedbc.jp/sparql

## `data`

```sparql
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX efo: <http://www.ebi.ac.uk/efo/>
PREFIX gwas: <http://rdf.ebi.ac.uk/terms/gwas/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX ro: <http://www.obofoundry.org/ro/ro.owl#>
PREFIX terms: <http://med2rdf.org/gwascatalog/terms/>
PREFIX tgv: <http://togovar.biosciencedbc.jp/variation/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

{{#if mode}}
SELECT DISTINCT ?tgv_id ?category ?label
{{else}}
SELECT ?category ?label (COUNT (DISTINCT ?tgv_id) AS ?count) 
{{/if}}
FROM <http://togovar.biosciencedbc.jp/variation>
FROM <http://togovar.biosciencedbc.jp/gwas-catalog>
FROM <http://togovar.biosciencedbc.jp/efo>
WHERE {
{{#if queryArray}}
  VALUES ?tgv_id { {{#each queryArray}} {{this}} {{/each}} }
{{/if}}
{{#if categoryArray}}
  {{#if mode}}
  VALUES ?category { {{#each categoryArray}} {{this}} {{/each}} } 
  {{else}}
  VALUES ?parent { {{#each categoryArray}} {{this}} {{/each}} }
  {{/if}}
{{/if}}

  GRAPH <http://togovar.biosciencedbc.jp/efo>{
{{#unless  mode}}
    ?category rdfs:subClassOf ?parent.
{{/unless}}
    ?category rdfs:label ?label.
    ?efo rdfs:subClassOf* ?category.
    ?efo rdf:type owl:Class.  # ?efo rdfs:subClassOf* ?category が?efoの値に関係なくtrueになってしまうため追加
  }

  GRAPH <http://togovar.biosciencedbc.jp/gwas-catalog>{
    ?assoc terms:mapped_trait_uri ?efo.
    ?assoc terms:dbsnp_url ?dbsnp.
    ?assoc terms:mapped_trait ?mapped_trait.
  }

  GRAPH <http://togovar.biosciencedbc.jp/variation>{
   ?dbsnp ^rdfs:seeAlso/dct:identifier ?tgv_id.
  } 
}
{{#if mode}}  
limit 100
{{else}}
ORDER BY DESC(?count)
{{/if}}
```

## `return`
```javascript
({mode, data})=>{
  const idVarName = "tgv_id";
  const idPrfix_mondo = "http://purl.obolibrary.org/obo/";
  const idPrfix_efo = "http://www.ebi.ac.uk/efo/";
  const categoryPrefix = "";

  if (mode == "objectList") return data.results.bindings.map(d=>{
    return {
      id: d[idVarName].value, 
      attribute: {
        categoryId: d.category.value.replace(idPrfix_mondo,"").replace(idPrfix_efo,""),
        uri: d.category.value,
        label : d.label.value.charAt(0).toUpperCase() + d.label.value.slice(1)   // 先頭の１文字だけを大文字にする。
      }
    }
  });

  if (mode == "idList") return Array.from(new Set(data.results.bindings.map(d=>d[idVarName].value.replace(idPrfix_mondo,"").replace(idPrfix_efo,"")))); 

  return data.results.bindings.map(d=>{ 
    return {
      categoryId: d.category.value.replace(idPrfix_mondo,"").replace(idPrfix_efo,""),
      label: d.label.value.charAt(0).toUpperCase() + d.label.value.slice(1),   // 先頭の１文字だけを大文字にする。
      count: Number(d.count.value),
      hasChild: (Number(d.count.value) > 1 ? true : false)  
    };
  });	
}
