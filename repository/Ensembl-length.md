# Ensemblエントリの長さで分類（原薗・八塚）

## Parameters
* `categoryIds` 必須パラメータ。0-99999といった長さの範囲を指定する。カンマ(",")で区切ることで複数の範囲を指定できる。もし指定がなければ、0-3000000の範囲を100,000で区切る
  * example: 0-10000,15000-20000
* `queryIds` オプションパラメータ。Ensembl gene IDの集合を指定できる。指定がない場合、全IDが検索の対象になる。カンマ(",")で区切ること。
  * example: ENSG00000006634,ENSG00000023909,ENSG00000040633,ENSG00000060339,ENSG00000060566,ENSG00000066044,ENSG00000067141,ENSG00000072736,ENSG00000073150,ENSG00000075886
* `mode`  必須パラメータ。出力を整形する。
  * example: idList, objectList

## `categoryList`
```javascript
({categoryIds}) => {
  const CG_RANGE = 20000;
  const CG_LIMIT = 400000;
  const CG_INFINITY = 9999999999;
  let retArray = [];
  categoryIds = categoryIds.replace(/" "/g,"").trim();
  if (categoryIds){
    const eachRange = categoryIds.split(",");
    eachRange.forEach(d=>{
      var parsedRange = d.split("-");
      retArray.push({min: parsedRange[0], max: parsedRange[1]});   
    });
  }else{
   let min = 0;
   while (min < CG_LIMIT){
     var max = min + CG_RANGE - 1;
     retArray.push({'min':min , 'max': max});
     min = max + 1;
   }
    retArray.push({"min": CG_LIMIT, "max": CG_INFINITY})
    retArray[0].min = 0;
  }
  return retArray;
}
```

## `id_filter`
- ユーザが指定した ID リストを配列に分割
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ").trim()
  if (queryIds.match(/[\s]/)){
    var ids = queryIds.split(/\s+/).map(id=>{
      if(id[3]=="G")
      {
        return "enssrc:" + id;
      }
      else
      {
        return "enssrct:" + id;
      }
    });
  }
  return ids;
  if (queryIds) return [queryIds];
  return false;
}

```

## Endpoint
 https://integbio.jp/rdf/ebi/sparql
 
## `query`
```sparql
# @endpoint https://integbio.jp/rdf/ebi/sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX biohack: <http://biohackathon.org/resource/faldo#>
PREFIX ensterm: <http://rdf.ebi.ac.uk/terms/ensembl/>
PREFIX enssrc: <http://rdf.ebi.ac.uk/resource/ensembl/>
PREFIX enssrct: <http://rdf.ebi.ac.uk/resource/ensembl.transcript/>
{{#if mode}}
SELECT DISTINCT ?ens_id ?bin_id
{{else}}
SELECT ?bin_id COUNT(?length) as ?count
{{/if}}
WHERE {
  SELECT DISTINCT ?ens_id ?bin_id ?length
  WHERE {
    {{#if id_filter}}
    VALUES ?ens { {{#each id_filter}} {{this}} {{/each}} }
   {{/if}}   
    VALUES ?func {ensterm:miRNA ensterm:lncRNA ensterm:protein_coding ensterm:pseudogene}
    VALUES (?bin_id ?min ?max) { {{#each categoryList}}("{{this.min}}-{{this.max}}" {{this.min}} {{this.max}}) {{/each}} } 
    ?ens obo:RO_0002162 taxon:9606 ;
          biohack:location ?location ;
          rdfs:label ?label;
          a ?func.

    ?location biohack:begin ?begin ;
              biohack:end ?end .
    ?begin a biohack:ExactPosition .
    ?begin biohack:position ?begin_pos . 
    ?end a biohack:ExactPosition .
    ?end biohack:position ?end_pos . 
    BIND(abs(?end_pos - ?begin_pos) as ?length)
    FILTER(?min <= ?length && ?length <= ?max)
  {{#if mode}}
    BIND(REPLACE(STR(?ens), "http://rdf.ebi.ac.uk/resource/ensembl", "") AS ?ens_id)
  {{/if}} 
 }
}
{{#unless mode}}
GROUP BY ?bin_id
ORDER BY ?bin_id
{{/unless}}
```

## `return`
```javascript
({mode, query})=>{
  if(mode){
    if(mode == "idList"){
      return query.results.bindings.map(d=>{
        return d.ens_id.value.replace(".transcript", "").replace("/", "");
      })
    }else if (mode == "objectList"){
      return query.results.bindings.map(d=>{
        var s1 = d.ens_id.value.replace(".transcript", "");
        var idStr = s1.replace("/", "");
        return {
          id: idStr, 
          attribute: {
            categoryId: d.bin_id.value, 
            label : d.bin_id.value.split("-")[0] + "-"
          }
        }
      });
    }
  }else{ //In case that mode is not defined
    var returnArrayBeforeSort = query.results.bindings.map(d=>{
      return {bin_id: d.bin_id.value, count: d.count.value, sortkey:parseInt(d.bin_id.value.split("-"))};
    });
    var returnArrayAfterSort = returnArrayBeforeSort.sort(function(a, b){return a.sortkey - b.sortkey})
    return returnArrayAfterSort.map(d=>{
      return {id: d.bin_id, label: d.bin_id.split("-")[0] + "-", count: d.count};
    })
  }
}
```