# Open TG-GATEsを化合物投与時の重症度スケールで分類する（信定）

## Endpoint

https://integbio.jp/rdf/sparql

## `count_grade`

```sparql
 #化合物を投与時の重症度スケールで分類
PREFIX ontology: <http://purl.jp/bio/101/opentggates/ontology/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT ?dose_grade  (count(distinct(?compound)) as ?count)
WHERE
{
  [ ontology:grade / rdfs:label ?grade_label ;
  ontology:hasSample ?sample;
 ontology:hasSample  [
    a ontology:Sample ;
    ontology:experimentalCondition [
      a ontology:ExperimentalCondition ;
      ontology:exposedCompound ?compound ;
      ontology:doseLevel ?dose_level
    ]]].
   ?compound  a ontology:ChemicalCompound .
  filter(LANG(?grade_label) = 'ja')
  BIND (concat(STR(?dose_level) , "_", STR(?grade_label)) as ?dose_grade)
}
ORDER by desc (?dose_grade)
```

## `return`

```javascript
({count_grade})=>{
  var list = count_grade.results.bindings;
  var r = [];
  var id_val = 0;
  for(var i = 0; i < list.length; i++){		
    r.push({id: id_val, label: list[i].dose_grade.value, count: list[i].count.value});
    id_val++;
  }	

  return r;
};	
```