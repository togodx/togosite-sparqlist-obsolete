# ChEMBL で薬を薬効（WHO ATC Code）で分類 (mode, hasChild対応, Number対応）（建石、山本）

* 入力：
  * ATCコードのカテゴリ。（デフォルトは全部：この場合、パラメータは空白)
* 出力：
  * 入力したATCコードのカテゴリのサブカテゴリに含まれるChEMBL Molecule ID数（サブカテゴリ単位で集計）
  * 一つの薬に複数のATCコードがついていることがありうる 

## Description

- Data sources
    - (More data sources description goes here..)
    - ATC: https://bioportal.bioontology.org/ontologies/ATC
    - ChEMBL-RDF 28.0: http://ftp.ebi.ac.uk/pub/databases/chembl/ChEMBL-RDF/
- Query
    - (More query details go here..)
    -  Input
        - ChEMBL ID
    - Output
        - ATC category ID
        - ATC category label

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters

* `categoryIds`   ATCコード、デフォルトは空。指定された場合、その下の内訳。
  *  第1階層：英大文字１ケタ, 第2階層：数字２ケタ, 第３階層：英大文字１ケタ, 第４階層：英大文字１ケタ, 第５階層：数字２ケタ （個別の薬）  
  * default:  
  * example: J (ANTIINFECTIVES FOR SYSTEMIC USE), J05 (ANTIVIRALS FOR SYSTEMIC USE), J05A (DIRECT ACTING ANTIVIRALS), J05AE (Protease inhibitors)
* `queryIds` ChEMBL IDのリスト、空白の場合は無条件 (2021/3/5「内部ID」形式にあわせて数字のみとした→2021/3/15 CHEMBL復活）
  * default:
  * example: CHEMBL17860,CHEMBL231779,CHEMBL231813,CHEMBL251634,CHEMBL292707,CHEMBL312862,CHEMBL43184,CHEMBL566315,CHEMBL600,CHEMBL63323
* `mode` (type: string)
  * example: idList, objectList

## `queryArray`
* ユーザが指定した ID リストを配列に分割

```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## `categoryArray`

category ID を配列に分割

```javascript
({categoryIds}) => {
  categoryIds = categoryIds.replace(/,/g," ")
  if (categoryIds.match(/[^\s]/)) return categoryIds.split(/\s+/);
  return false;
}
```

## `data`
* ATCコードの文字列が階層的になっていることに頼る（by 山本さん）
* Todo:categoryIdがコードの書式にあっているかのチェック（あっていない場合に何を返すか？）
* Endpointが https://integbio.jp/rdf/mirror/ebi/sparql のとき
  FROM <http://rdf.ebi.ac.uk/dataset/chembl>
* Endpointが https://integbio.jp/togosite/sparql のとき
  FROM <http://rdf.integbio.jp/dataset/togosite/chembl>    

```sparql
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#> 
PREFIX molecule: <http://rdf.ebi.ac.uk/resource/chembl/molecule/>
{{#if mode}}
SELECT DISTINCT ?atctop ?molecule
{{else}}
SELECT ?atctop (count(DISTINCT ?molecule) as ?count)
{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>  

WHERE {
  {{#if queryArray}}
      VALUES ?molecule { {{#each queryArray}} molecule:{{this}} {{/each}} }
  {{else}}                                                
      # 無条件      
  {{/if}}
  
  ?molecule cco:atcClassification ?atc .
        
  {{#if categoryArray}}
    VALUES ?parent { {{#each categoryArray}} "{{this}}" {{/each}} }
    VALUES (?plen ?clen) { (1 3) (3 4) (4 5) (5 7) }
    FILTER(STRLEN(?parent) = ?plen)  
    FILTER strstarts(?atc, ?parent)  
    {{#if mode}}   
      BIND (substr(?atc, 0, ?plen) as ?atctop)  
    {{else}}
      BIND (substr(?atc, 0, ?clen) as ?atctop)  
    {{/if}}
  {{else}}    
      BIND (substr(?atc, 0, 1) as ?atctop)  
  {{/if}}

}
GROUP BY ?atctop
ORDER BY ?atctop


```

## `atcArray` 
```javascript
({data})=>{
  var a=data.results.bindings.map(d=>{return(d.atctop.value)  });	
  return(Array.from(new Set(a)))
}
```


## `labeldata`
* ATCコードのラベル取得
* Endpointが https://integbio.jp/rdf/mirror/bioportal/sparql のとき
  FROM <http://integbio.jp/rdf/mirror/bioportal/atc>

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX atc: <http://purl.bioontology.org/ontology/UATC/>

SELECT ?atc ?label 
FROM <http://rdf.integbio.jp/dataset/togosite/atc>
WHERE 
{
    VALUES ?atc  { {{#each atcArray}} atc:{{this}}  {{/each}} }
    ?atc  skos:prefLabel ?label.    
}
```


## `return` 
```javascript
({data,labeldata,mode})=>{
  const uatcprefix="http://purl.bioontology.org/ontology/UATC/"
  const moleculeprefix="http://rdf.ebi.ac.uk/resource/chembl/molecule/"
  var labels={}
  var r=[]
  
  labeldata.results.bindings.forEach(d=>{
    var atccode=d.atc.value.replace(uatcprefix,"")
    labels[atccode]=d.label.value+" ("+atccode+")"
  });	
  if (mode === "idList") {
    return Array.from(new Set(
      data.results.bindings.map((d) =>
        d.molecule.value.replace(moleculeprefix, "")
      )
    ));
  } else if (mode === "objectList") {  
    data.results.bindings.forEach(d=>{
      r.push({
        id:d.molecule.value.replace(moleculeprefix, ""),
        attribute: {
          categoryId:d.atctop.value,
          label:labels[d.atctop.value],
          uri:uatcprefix+d.atctop.value
        }
      })
    });
  } else {
    data.results.bindings.forEach(d=>{
      var a = d.atctop.value;
      r.push({
        categoryId:a,
        label:labels[a],
        count:Number(d.count.value),
        hasChild:(a.length<7) //コードの文字数が固定で最下層だと7であることを利用	
      })
    });
  }
  return (r)
  
}  
```
