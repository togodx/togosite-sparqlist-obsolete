# variant GWAS (Genome-wide association study) (三橋)

## Description

- Data sources
    -  [TogoVar](https://togovar.biosciencedbc.jp/?) (limited to variants with frequency data in Japanese populations)
- Query
    - Input
        - TogoVar id
    - Output
        -  [Mapped trait used in GWAS Catalog](https://www.ebi.ac.uk/gwas/docs/ontology)

## Parameters

* `categoryIds` (type: Mapped trait represented in Experimental factor ontology (EFO))
  * default: EFO_0000001
  * example: MONDO_0020683,Orphanet_68335,EFO_0000001,EFO_0000408
* `queryIds` (type:TogoVar)
  * example: tgv704775,tgv704941,tgv772580,tgv246970,tgv39969772,tgv40054079,tgv42043030
* `mode`
  * example: idList, objectList
* `endpoint` Endpoint
  * default: https://integbio.jp/togosite/sparql

## categoryIdsのdefault値の選択について
- categoryIdsのデフォルトを[EFO_0000001(EFOのトップ)](https://www.ebi.ac.uk/ols/ontologies/efo/terms?iri=http%3A%2F%2Fwww.ebi.ac.uk%2Fefo%2FEFO_0000001&viewMode=All&siblings=false)と[EFO_0000408(diseaseのトップ)](https://www.ebi.ac.uk/ols/ontologies/efo/terms?iri=http%3A%2F%2Fwww.ebi.ac.uk%2Fefo%2FEFO_0000408)にするかでメリットデメリットがある。
  - EFO_0000001の場合：
    - メリット：カバーできるGWAS Catalogのvariant数(123,485,[検証用SPARQL](https://is.gd/tFovng))が多い。
    - デメリット：疾患を探すのが少し難しい。
  - EFO_0000408の場合：
    - メリット：疾患の階層がトップに表示される
    - メリット：カバーできるGWAS Catalogのvariant数(27,674,[検証用SPARQL](https://is.gd/QQP2Ri))少ない。
  - 結論：experimental factor (EFO_0000001)をトップとする。
    - 理由：EFO_0000408(disease)のサブカテゴリでないGWAS catalogに含まれるEFOエントリ（ [SPARQLで検索](https://is.gd/GTHjwB) ）には、
   　information entity > measurementの下にあるEFO_0009282("sodium measurement"), EFO_0009718("peak expiratory flow")など重要なヒト関連形質が含まれるため。

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
  return (categoryIds == "" ? false :  categoryIds.split(/\s+/).map(categoryId=>categoryId.replace("EFO_","efo:EFO_").replace("MONDO_","obo:MONDO_").replace("Orphanet_","ordo:Orphanet_").replace("BFO_","obo:BFO_").replace("IAO_","obo:IAO_")))
 }
```

## Endpoint

https://integbio.jp/togosite/sparql

## `data`

```sparql
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX efo: <http://www.ebi.ac.uk/efo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX ordo: <http://www.orpha.net/ORDO/>
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
FROM <http://rdf.integbio.jp/dataset/togosite/variation>
FROM <http://rdf.integbio.jp/dataset/togosite/gwas-catalog>
FROM <http://rdf.integbio.jp/dataset/togosite/efo>
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

  GRAPH <http://rdf.integbio.jp/dataset/togosite/efo>{
{{#unless  mode}}
    ?category rdfs:subClassOf ?parent.
{{/unless}}
    ?category rdfs:label ?label.
    ?efo rdfs:subClassOf* ?category.
    ?efo rdf:type owl:Class.  # ?efo rdfs:subClassOf* ?category が?efoの値に関係なくtrueになってしまうため追加
  }

  GRAPH <http://rdf.integbio.jp/dataset/togosite/gwas-catalog>{
    ?assoc terms:mapped_trait_uri ?efo.
    ?assoc terms:dbsnp_url ?dbsnp.
    ?assoc terms:mapped_trait ?mapped_trait.
  }

  GRAPH <http://rdf.integbio.jp/dataset/togosite/variation>{
   ?dbsnp ^rdfs:seeAlso/dct:identifier ?tgv_id.
  } 
}
{{#unless mode}}  
ORDER BY ?label
{{/unless}}
```

## `return`
```javascript
({mode, data})=>{
  const idVarName = "tgv_id";
  const idPrfix_obo = "http://purl.obolibrary.org/obo/";
  const idPrfix_efo = "http://www.ebi.ac.uk/efo/";
  const idPrfix_ordo="http://www.orpha.net/ORDO/";
  const categoryPrefix = "";

  if (mode == "objectList") return data.results.bindings.map(d=>{
    return {
      id: d[idVarName].value, 
      attribute: {
        categoryId: d.category.value.replace(idPrfix_obo,"").replace(idPrfix_efo,"").replace(idPrfix_ordo,""),
        uri: d.category.value,
        label : d.label.value.charAt(0).toUpperCase() + d.label.value.slice(1)   // 先頭の１文字だけを大文字にする。
      }
    }
  });

  if (mode == "idList") return Array.from(new Set(data.results.bindings.map(d=>d[idVarName].value.replace(idPrfix_obo,"").replace(idPrfix_efo,"").replace(idPrfix_ordo,"")))); 

  return data.results.bindings.map(d=>{ 
    return {
      categoryId: d.category.value.replace(idPrfix_obo,"").replace(idPrfix_efo,"").replace(idPrfix_ordo,""),
      label: d.label.value.charAt(0).toUpperCase() + d.label.value.slice(1),   // 先頭の１文字だけを大文字にする。
      count: Number(d.count.value),
      hasChild: (Number(d.count.value) > 1 ? true : false)  
    };
  });	
}
