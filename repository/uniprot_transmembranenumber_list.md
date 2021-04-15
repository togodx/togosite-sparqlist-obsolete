# (作業中)uniprotのエントリを膜貫通回数で分類_genelist (井手, 守屋)

## Parameters
* `categoryId` (type: 膜貫通部位数)
  * example: 4,8
* `queryIds` (type: uniprot)
  * example: Q5VV42,Q9BSA9,Q12884,P04233,O15162,O00322,P16070,O75844,Q9BXK5,Q12983,P08195,Q496J9,P63027,P51681,P58335,Q9Y5U4,P12830,P08581,Q96NB2,O75746,Q9Y548

## `query_array`

```javascript
({queryIds}) => {
  queryIds = queryIds.replace(/,/g," ")
  if (queryIds.match(/[^\s]/)) return queryIds.split(/\s+/);
  return false;
}
```

## `category_array`

```javascript
({categoryId}) => {
  categoryId = categoryId.replace(/,/g," ");
  if (categoryId.match(/[^\s]/)) return categoryId.split(/\s+/);
  return false;
}
```

## Endpoint

https://integbio.jp/rdf/mirror/uniprot/sparql

## `transmembrane`

```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX upid: <http://purl.uniprot.org/uniprot/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>

SELECT ?number (COUNT(DISTINCT ?uniprot) AS ?count)
WHERE{
	SELECT ?uniprot (COUNT(DISTINCT ?annotation) AS ?number)
	WHERE {
        {{#if querry_array}}
        VALUES ?uniprot { {{#each query_array}} upid:{{this}} {{/each}} }
        {{/if}}        
	  	?uniprot a up:Protein ;
  		         up:organism taxon:9606 ;
  		         up:proteome ?proteome ;
  		         up:annotation ?annotation .       
  		?annotation rdf:type up:Transmembrane_Annotation .
  	FILTER(REGEX(STR(?proteome), "UP000005640"))
    }
}    
ORDER BY ?number
```

## `zero`
- 膜貫通部位を持たないタンパク質の数
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX taxon: <http://purl.uniprot.org/taxonomy/>
SELECT (COUNT(DISTINCT ?uniprot) AS ?count)
WHERE {
  {{#if querry_array}}
  VALUES ?uniprot { {{#each query_array}} upid:{{this}} {{/each}} }
  {{/if}}
  ?uniprot a up:Protein ;
           up:organism taxon:9606 ;
           up:proteome ?proteome .
  FILTER(REGEX(STR(?proteome), "UP000005640"))
  MINUS {
    ?uniprot up:annotation [ a up:Transmembrane_Annotation ] .
  }
}
```

## `results`

```javascript
({category_array, transmembrane, zero})=>{ 
  var r = [];
  transmembrane.results.bindings.unshift( {count: {value: zero.results.bindings[0].count.value}, number: {value: 0}}  ); // カウント 0 を追加 
  transmembrane.results.bindings.forEach(d=>{
    var num = d.number.value;
    if (category_array){
      if (category_array.includes(num)){
      	r.push({categoryId: num, label: num, count: d.count.value});
      }
  	}else{
      r.push({categoryId: num, label: num, count: d.count.value});
    }
  });
  return r;
};	
```
