# knapsack  (信定)
- 生物種ーknapsack compoundの分類 
- kingdom-family-oraganism-compound

## Endpoint
https://mb2.ddbj.nig.ac.jp/sparql

## `leaf`
- organismとcompound のアノテーション関係
```sparql
prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
prefix metabo: <http://ddbj.nig.ac.jp/ontolofies/metabobank/>
select distinct ?compound_id ?tax_id ?organism ?family ?kingdom
from <http://mb-wiki.nig.ac.jp/resource>
where {
?s  a metabo:KNApSAcKCoreAnnotations ;
rdfs:seeAlso ?taxonomy;
metabo:family ?family 
metabo:kingdom ?kingdom ;
metabo:organism ?organism .
BIND (substr(str(?s ), 35, 9) AS ?compound_id)
BIND (strafter(str(?taxonomy), "http://identifiers.org/taxonomy/") AS ?tax_id)
}
```

