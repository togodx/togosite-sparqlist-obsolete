# Open TG-GATEsを投与経路ごとの実験数で分類する（信定）

## Endpoint

https://integbio.jp/rdf/sparql

## `count_adm_uniset`

```sparql
#投与経路ごとの実験数(Uniset=a set of control and test sample)を数える
#memo: Gavage=強制経口投与
PREFIX ontology: <http://purl.jp/bio/101/opentggates/ontology/>
SELECT  ?adm_route (count(distinct?UnitSet) as ?count)
WHERE
{
  ?UnitSet a ontology:UnitSet .
  ?UnitSet   ontology:hasControlSample | ontology:hasTestSample ?sample .
  ?sample a ontology:Sample ;
          ontology:experimentalCondition ?ExperimentalCondition .
  ?ExperimentalCondition a ontology:ExperimentalCondition ;
                         ontology:admRoute ?adm_route .
}
order by desc (?count)
```

## `return`

```javascript
({count_adm_uniset})=>{
  var list = count_adm_uniset.results.bindings;
  var r = [];
  var id_val = 0;
  for(var i = 0; i < list.length; i++){		
    r.push({id: id_val, label: list[i].adm_route.value, count: list[i].count.value});
    id_val++;
  }	

  return r;
};	
```
