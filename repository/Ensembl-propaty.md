# 指定されたEnsemblエントリのプロパティを表示する

# TogoSite query template

- Parameters
  - queryIds (Optional)
## Parameters

* `queryIds` 必須。Ensembl gene IDの集合。カンマ","で区切ること。
  * example: ENSG00000006634,ENSG00000023909,ENSG00000040633,ENSG00000060339,ENSG00000060566,ENSG00000066044,ENSG00000067141,ENSG00000072736,ENSG00000073150,ENSG00000075886


## `queryIds`
- ユーザが指定した ID リストを配列に分割
```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ").trim()
  if (queryIds.match(/[\s]/)) return queryIds.split(/\s+/);
  if (queryIds) return [queryIds];
  return false;
}
```

## Endpoint
 https://integbio.jp/rdf/ebi/sparql
 
## `propaties`
```sparql
# @endpoint https://integbio.jp/rdf/ebi/sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX identifiers: <http://identifiers.org/>
PREFIX biohack: <http://biohackathon.org/resource/faldo#>
PREFIX ensterm: <http://rdf.ebi.ac.uk/terms/ensembl/>
PREFIX enssrc: <http://rdf.ebi.ac.uk/resource/ensembl/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
SELECT DISTINCT ?idn as ?Ensembl_gene_id ?function_category ?length ?max
WHERE {
  {{#if category}}
  VALUES ?function_category { {{#each category}} ensterm:{{this}} {{/each}} }
  {{/if}}
  ?ensg obo:RO_0002162 taxon:9606 ;
        biohack:location ?location ;
        rdfs:label ?label;
        a ?function_category;
        dc:identifier ?idn.
  {{#if queryIds}}
  VALUES ?queryIds_area { {{#each queryIds}} enssrc:{{this}} {{/each}} }
  FILTER(?ensg = ?queryIds_area)
  {{/if}}

  ?location biohack:begin ?begin ;
            biohack:end ?end .
  ?begin a biohack:ExactPosition .
  ?begin biohack:position ?begin_pos . 
  ?end a biohack:ExactPosition .
  ?end biohack:position ?end_pos . 
  BIND(abs(?end_pos - ?begin_pos) as ?length)
  {{#if max_length}}
  BIND({{max_length}} as ?max)
  #VALUES ?max {max_length}
  FILTER (xsd:integer(?length) < xsd:integer(?max))
  {{/if}}
  {{#if min_length}}
  BIND({{min_length}} as ?min)
  FILTER (xsd:integer(?length) > xsd:integer(?min))
  {{/if}}
}
ORDER BY DESC(?length)
```

## `return`
```javascript
({length})=>{
  var list = length.results.bindings;
  var r = [];
  for(var i = 0; i < list.length; i++){		
    r.push({id: list[i].Ensembl_gene_id.value, function_category: list[i].function_category.value.split("/").pop(), length: list[i].length.value});
  }	
  return r;
};	
```