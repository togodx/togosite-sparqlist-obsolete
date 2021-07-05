# exon数ごとにトランスクリプト数をカウント（鈴木・八塚）

## Description

- Data sources
    - Ensembl human release 102: [http://nov2020.archive.ensembl.org/Homo_sapiens/Info/Index](http://nov2020.archive.ensembl.org/Homo_sapiens/Info/Index)
- Input/Output
    -  Input
        - Ensembl Transcription ID -> queryIds
        - "(Number of exons)" or range ("(Min number of exons)-(Max number of exons)") -> categoryIds
    - Output
        - Output1("mode": none): List of number of exons for each category
        - Output2("mode": idList): List of Ensembl Transcription ID
        - Output3("mode": objectList): List of Ensembl Transcription ID and attribute of category
- Supplementary information
    - This is the result of counting the number of exons for each transcript of each gene.
    - 各遺伝子の転写産物ごとにエクソン数をカウントした結果です。
        
## Endpoint

https://integbio.jp/togosite/sparql

## Parameters

* `categoryIds` exon数を"(最小値)-(最大値)"または特定の数字で指定する。カンマ区切りで複数の範囲を指定できる。
  * example:1, 2-10
* `queryIds` 必須パラメータ。デフォルトは空。数える対象の ID リスト。ユーザが ID のリストを指定した場合、全体の内訳の代わりに、ユーザの ID が各内訳に何個ずつ該当するかを返す。空白文字かコンマ区切りのリスト
  * example: ENST00000589042,ENST00000342175,ENST00000591111,ENST00000604864,ENST00000341594,ENST00000358025,ENST00000460472
* `mode` 必須パラメータ。内訳の代わりに該当する ID のリストを返す（デフォルトはオフ）idList: リストだけ、objectList: Attributeの入ったリスト（Attribute は下階層ではなく、categoryid で指定したカテゴリ）
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
  const RANGE = 10;
  const CG_COUNT = 21;
  const LIMIT = 10000;
  let index = 0;
  let cArray = [];
  categoryIds = categoryIds.replace(/,/g," ");
  if (categoryIds.match(/[^\s]/)){
    var rArray = categoryIds.split(/\s+/);
    rArray.forEach(d=>{
      var minmax = d.split(/-/);
      if (minmax[1]){
        cArray.push({'min':minmax[0], 'max':minmax[1]});
      }else{
        cArray.push({'min':minmax[0], 'max':minmax[0]});
      }
      index++;
    });
  }else{
    cArray.push(  {"min": 1, "max": 1},
                  {"min": 2, "max": 2},
                  {"min": 3, "max": 3},
                  {"min": 4, "max": 4},
                  {"min": 5, "max": 5},
                  {"min": 6, "max": 6},
                  {"min": 7, "max": 7},
                  {"min": 8, "max": 8},
                  {"min": 9, "max": 9},
                  {"min": 10, "max": 10},
                  {"min": 11, "max": 15},
                  {"min": 16, "max": 20},
                  {"min": 21, "max": 50},
                  {"min": 51, "max": 100},
                  {"min": 101, "max": 200},
                  {"min": 201, "max": 400},
                  {"min": 401, "max": 1000},
                  {"min": 1001,"max": 10000});
    //let pos = 1;
    //for (let i=0;i < CG_COUNT;i++){
    //  cArray.push({'min':pos, 'max':pos+RANGE-1});
    //  pos += RANGE;
    //}
    //cArray[0]['min'] = 0;
    //cArray[CG_COUNT-1]['max'] = LIMIT;
  }
  return cArray;
}
```

## `query`
```sparql
# @endpoint https://integbio.jp/rdf/ebi/sparql
PREFIX obo:<http://purl.obolibrary.org/obo/>
PREFIX taxon:<http://identifiers.org/taxonomy/>
PREFIX enst:<http://rdf.ebi.ac.uk/resource/ensembl.transcript/>
PREFIX terms:<http://rdf.ebi.ac.uk/terms/ensembl/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>

{{#if mode}}
SELECT DISTINCT ?enst_id ?bin_id
{{else}}
SELECT ?bin_id (COUNT(DISTINCT ?enst) AS ?count)
{{/if}}       
WHERE {
  VALUES (?bin_id ?min ?max) { {{#each categoryArray}}("{{this.min}}-{{this.max}}" {{this.min}} {{this.max}}) {{/each}} }
  FILTER(?min <= ?exon_count &&  ?exon_count <= ?max){
     SELECT ?enst (COUNT(DISTINCT ?exon) AS ?exon_count)
      WHERE {
        {{#if queryArray}}
          VALUES ?enst { {{#each queryArray}} enst:{{this}}{{/each}} }
        {{/if}}
        ?enst obo:SO_has_part ?exon;
              obo:SO_transcribed_from ?ensg .
        ?ensg obo:RO_0002162 taxon:9606 ; # in taxon
              faldo:location ?ensg_location .
        BIND (strbefore(strafter(str(?ensg_location), "GRCh38/"), ":") AS ?chromosome)
        VALUES ?chromosome {
          "1" "2" "3" "4" "5" "6" "7" "8" "9" "10"
          "11" "12" "13" "14" "15" "16" "17" "18" "19" "20" "21" "22"
          "X" "Y" "MT"
        }
    }GROUP BY ?enst
  }
{{#if mode}}
  BIND(REPLACE(STR(?enst), "http://rdf.ebi.ac.uk/resource/ensembl.transcript/", "") AS ?enst_id)
{{/if}}  
}
{{#unless mode}}
GROUP BY ?bin_id
ORDER BY ?bin_id
{{/unless}}
```

## `results`

```javascript
({mode, query})=>{
    if (mode == "idList"){
        return query.results.bindings.map(d=>{
          return d.enst_id.value;
        });
    }else if (mode == "objectList"){
      return query.results.bindings.map(d=>{
        var range = d.bin_id.value.split("-");  //
        return {
          id: d.enst_id.value, 
          attribute: {
            categoryId: d.bin_id.value, 
            label : makeLabel(range[0], range[1])   //d.bin_id.value
          }
        }
      });
    }else{
      let rArray = query.results.bindings.map(d=>{
        var range = d.bin_id.value.split("-");
        return {
          id: d.bin_id.value,
          min: range[0],
          label: makeLabel(range[0], range[1]),
          count: Number(d.count.value)
        }
      });
      rArray.sort(function(a, b){
        var aValue = parseInt(a.min);
        var bValue = parseInt(b.min);
        if(aValue < bValue) return -1;
    	if(aValue > bValue) return 1;
    	return 0;
      });
      
      return rArray.map(d=>{
        return {
          categoryId: d.id,
          label: d.label,
          count: Number(d.count)
        }
      });
    }
       function makeLabel(num1, num2) {
         if (num1 == num2) {
           return num1;
         } else {
           return num1+"-"+num2 ;
         }
       }
    
};	
```