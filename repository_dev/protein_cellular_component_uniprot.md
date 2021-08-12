# Gene Ontology 'Cellular component' 内訳（守屋）

- Required SPARQLet: uniprot_keywords

## Description

- Data sources
    - [UniProt](https://www.uniprot.org/)
    
- Query
    - Input
        - UniProt ID
    - Output
        - Gene Ontology terms of cellular component domain
        
## Parameters

* `categoryIds` (type: UniProt keyword ID) (Req.) 
  * default: GO_0005575
* `queryIds` (type: UniProt)
  * example: Q9NYF8,Q4V339,A6NCE7,A7E2F4,P69849,A6NN73,Q92928,Q5T1J5,P0C7P4,Q6DN03,P09874,Q08211,Q5T4S7,P12270,Q9UPN3,P07814,P53621,P49321,P0C629,Q9BZK8,Q9BY65
* `mode`
  * example: idList, objectList
  
## `httpreq`

```javascript
async ({queryIds, categoryIds, mode})=>{
  let url = "uniprot_go"; // parent SPARQLet relative path
  let body = "categoryIds=" + categoryIds;
  if (queryIds) body += "&queryIds=" + encodeURIComponent(queryIds);
  if (mode ) body += "&mode=" + mode;
  let options = {
    method: 'POST',
    body	: body,
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  return await fetch(url, options).then(res=>res.json());
}
```
  