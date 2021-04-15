# PubChem でHydrogen bond Donor Count 10ごとに区切りカウント（FDA Approved Drugs）

## Parameters

* `filter` Pubchem Compound ID (番号のみ）のリスト、なければ FDA Approved Drugs であるもの全部
  * default:
  * example: 2793, 6037, 4724, 37632, 4823, 5217, 54120, 42008, 10235, 17676, 5523, 4116  
* `interval` いくつごとに区切るか
  * default: 10
  * example: 5 10 100

## Endpoint

https://integbio.jp/rdf/pubchem/sparql


## `filter_list`

```javascript
({filter}) => {
  filter = filter.replace(/\s/g,"")
  if (filter) return filter.split(",");
  return false;
}
```


## `hydrogen_bond_donor_count`

```sparql
## https://integbio.jp/rdf/pubchem/sparql
## FDA approved drugs -> hydrogen bond donor count


PREFIX sio: <http://semanticscience.org/resource/>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX compound: <http://rdf.ncbi.nlm.nih.gov/pubchem/compound/CID>
PREFIX pubchemv: <http://rdf.ncbi.nlm.nih.gov/pubchem/vocabulary#>

PREFIX xsd: <http://www.w3.org/2001/XMLSchema#> 
SELECT distinct ?cid ?donor_count
WHERE {
{{#if filter_list}}
  VALUES ?cid { {{#each filter_list}} compound:{{this}} {{/each}} }
{{else}}
  ?cid      obo:has-role pubchemv:FDAApprovedDrugs .
{{/if}} 
  ?cid      sio:has-attribute ?attr.  
  ?attr a sio:CHEMINF_000387 . 
  ?attr sio:has-value ?donor_count.
} 
                 

```
                 
                 
## `return`

```javascript
({hydrogen_bond_donor_count, interval})=>{
  var list = hydrogen_bond_donor_count.results.bindings;
  var r = [];
  var tmp = {}
  var id_val;
  var v;
  
  for(var i = 0; i < list.length; i++){	
    v = list[i].donor_count.value
    id_val = Math.floor(v / interval) * interval
    if (tmp[id_val]){
    	tmp[id_val]++
    } else{
    	tmp[id_val] = 1
    }
  }	
  for (var k in tmp) { 
    var kk = Number(k) + Number(interval -1 )
    r.push({id: k, label: k.toString() + "-" +  kk.toString() , count: tmp[k]});  
  }
  return r;
};	
```
                 
                 