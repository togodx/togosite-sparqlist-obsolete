# Genes expressed in tissues (Bgee) (池田, 守屋)

## Description

- Data sources
    - Bgee latest version: [https://bgee.org/?page=sparql](https://bgee.org/?page=sparql)

## Endpoint

https://integbio.jp/togosite/sparql

## `graph`
```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
SELECT DISTINCT ?child ?child_label ?parent ?parent_label
FROM <http://rdf.integbio.jp/dataset/togosite/uberon>
WHERE {
  ?parent rdfs:label ?parent_label ;
     ^rdfs:subClassOf ?child .
   ?child rdfs:label ?child_label .
  ?parent rdfs:subClassOf* obo:BFO_0000004 .
}
LIMIT 1
```

## `return`
```javascript
({graph})=>{
  const fetchReq = async (url, options, body)=>{
    if (body) options.body = body;
    return await fetch(url, options).then(res=>res.json());
  }
  
  const url = "test_gene_expressed_tissue_bgee_backend";
  let options = {
    method: 'POST',
    headers: {
      'Accept': 'application/json',
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  }
  
  let tree = [
    {
      id: "BFO_0000004",
      label: "independent continuant"
    }
  ]
  
  let onto = {};
  graph.results.bindings.map(async d => {
    let anat = d.child.value.replace("http://purl.obolibrary.org/obo/", "");
    anat = "UBERON_0001296";
    if (! onto[anat]) {
      onto[anat] = true;
      tree.push({
        id: anat,
        label: d.child_label.value,
        parent: d.parent.value.replace("http://purl.obolibrary.org/obo/", "")
      });
      let json = await fetchReq(url, options, "obo=" + anat);
      if (json[0]) tree = tree.concat(json);
    }
  });
  return tree;
};
```
