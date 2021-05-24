# PubChem で薬を薬効で分類（FDA Approved Drugs → WHO ATC Code）(フロント開発用 with hasChild flag.)（建石, 守屋, 山本）

*  atc_classification_haschild （objectList はOK）を idList対応に修正 (2021/3/24)
*  Filterの効率化 (2021/4/2)

## Description 
* Data sources
	* PubChem-RDF: ftp://ftp.ncbi.nlm.nih.gov/pubchem/RDF/ （Version 2021-03-01 ） 
      * https://integbio.jp/rdf/dataset/pubchem より、
      ChEMBLまたはChEBIとリンクされているノードに関するデータのみを取得

* Query
	* Input
  		*  categoryIds:  ATC code (https://www.whocc.no/atc_ddd_index/)
    	   * デフォルトは空白。
    	   *  第1階層：英大文字１ケタ, 第2階層：数字２ケタ, 第３階層：英大文字１ケタ, 第４階層：英大文字１ケタ, 第５階層：数字２ケタ （個別の薬）  
  		* queryIds
    	  *  PubChem Compound ID (数字のみ）、空白の場合はFDA Approved Drugsであるもの全体
        * mode  
  * Output
    * modeが空の場合
      * 入力したATCコードで示されるカテゴリのサブカテゴリに含まれるPubChem Compound ID数（サブカテゴリ単位で集計）
      *  categoryIdsが空の場合は、第1階層の内訳
      * 一つの薬に複数のATCコードがついていることがありうるがその場合は重複して数えられる 
    * modeがidListの場合
      * categoryIdsで指定されたカテゴリに属する物質のPubChem Compound ID
    * modeがobjectListの場合
      * categoryIdsで指定されたカテゴリに属する物質のPubChem Compound ID、ATCコード、ラベル


## Endpoint

https://integbio.jp/togosite/sparql

## Parameters

* `categoryIds`   ATCコード、デフォルトは空。指定された場合、その下の内訳。
  * default:  
  * example: J (ANTIINFECTIVES FOR SYSTEMIC USE), J05 (ANTIVIRALS FOR SYSTEMIC USE), J05A (DIRECT ACTING ANTIVIRALS), J05AE (Protease inhibitors)
* `queryIds` PubChem Compound ID (数字のみ）、空白の場合はFDA Approved Drugsであるもの全体
  * default: 
  * example: 3561, 6957673, 3226, 452548, 19861, 41781, 4909, 15814656, 13342, 11597698, 3396, 60937, 86767262, 43507, 3342, 4642, 5311497, 3356, 37464, 5353853 
* `mode`
  * example: idList, objectList

## `queryArray`
- ユーザが指定した ID リストを配列に分割
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## `categoryArray`
- category ID を配列に分割
```javascript
({categoryIds}) => {
  categoryIds = categoryIds.replace(/,/g," ")
  if (categoryIds.match(/[^\s]/)) return categoryIds.split(/\s+/);
  return false;
}
```

## `data`

```sparql
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX compound: <http://rdf.ncbi.nlm.nih.gov/pubchem/compound/CID>
PREFIX concept: <http://rdf.ncbi.nlm.nih.gov/pubchem/concept/ATC_>
PREFIX pubchemv: <http://rdf.ncbi.nlm.nih.gov/pubchem/vocabulary#>
PREFIX dcterms: <http://purl.org/dc/terms/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
{{#if mode}}
SELECT DISTINCT ?cid ?category ?label
{{else}}
SELECT ?category ?label ?count (SAMPLE (?child_atc) AS ?child)
WHERE {
  {
    SELECT ?category ?label (COUNT (DISTINCT ?cid) AS ?count)
{{/if}}
    WHERE {
      {{#if queryArray}}
      VALUES ?cid { {{#each queryArray}} compound:{{this}} {{/each}} }
        
      {{/if}}
      {{#if categoryArray}}
        {{#if mode}}
      VALUES ?category
        {{else}}
      VALUES ?parent
        {{/if}}
          { {{#each categoryArray}} concept:{{this}} {{/each}} }
      {{/if}}
 
      ?attr a sio:CHEMINF_000562 ;
            sio:is-attribute-of ?cid ; 
            sio:has-value  ?WHO_INN ;
            dcterms:subject ?WHO_ATC .
      ?WHO_ATC skos:broader* ?category ;
  			   a skos:concept .
      # filter(strstarts(str(?WHO_ATC), "http://rdf.")) # MeSH IDが入っていることがあるのを捨てる。
                                                      # ATCコード  http://rdf.ncbi.nlm.nih.gov/pubchem/concept/ATC_xxxx
                                                      # MeSH ID    http://id.nlm.nih.gov/mesh/Mxxxxxx
      
      {{#if categoryArray}}
        {{#unless mode}}
      ?category skos:broader  ?parent.
        {{/unless}}
      {{else}}                     
          filter(strlen(str(?category)) = 49)     # regex(str(?category),http://rdf.ncbi.nlm.nih.gov/pubchem/concept/ATC_[A-Z]$) 
      {{/if}}  
      ?category skos:prefLabel  ?label .
{{#unless mode}}
    } group by ?label ?category
  } 
  OPTIONAL { # 内訳時の hasChild のフラグ用 （サブクエリでくくらないと、FILTERとOPTIONALでバグる。mode=*Listのときはサブクエリにしない）
    ?child_atc skos:broader ?category .
  }
{{/unless}}
} ORDER BY ?category

```

## `return`
- 整形
```javascript
({mode, data})=>{
  const idVarName = "cid";
  const idPrfix = "http://rdf.ncbi.nlm.nih.gov/pubchem/compound/CID";
  const categoryPrefix = "http://rdf.ncbi.nlm.nih.gov/pubchem/concept/ATC_";
  if (mode == "objectList") {
    return data.results.bindings.map(d=>{
      var categorycode = d.category.value.replace(categoryPrefix, "")
      return {
        id: d[idVarName].value.replace(idPrfix, ""), 
        attribute: {
          categoryId: d.category.value.replace(categoryPrefix, ""), 
          uri: d.category.value,
          label : capitalize(d.label.value)+" ("+categorycode+")"
        }
      }
    });
  } else if (mode == "idList") {
    var r = data.results.bindings.map(d=>{
      return d[idVarName].value.replace(idPrfix, "")
    })
    return Array.from(new Set(r))
  } else {  
    return data.results.bindings.map(d=>{
      var categorycode = d.category.value.replace(categoryPrefix, "")
      return {
        categoryId: categorycode, 
        label: capitalize(d.label.value)+" ("+categorycode+")",
        count: Number(d.count.value),
        hasChild: Boolean(d.child)
      };
    });
  }
  function capitalize(str) {
    return str.charAt(0).toUpperCase() + str.substring(1);
  }
}
```