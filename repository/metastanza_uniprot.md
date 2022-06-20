# Protein attributes from Uniprot RDF（井手, 守屋, 池田）

## Parameters
* `uniprot`
  * default: P06493
  * example: P06493

## Endpoint
https://integbio.jp/togosite/sparql

## `main`
一つの SPARQL だと遅いので分割
```sparql
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX core: <http://purl.uniprot.org/core/>
PREFIX db: <http://purl.uniprot.org/database/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT ?entry ?id ?mnemonic ?full_name ?short_name ?length ?mass 
  (GROUP_CONCAT(distinct ?pdb ; separator = ",") AS ?pdbs)
  (COUNT(DISTINCT ?citation) AS ?citation_number)
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE{
  VALUES ?entry { uniprot:{{uniprot}} }
  ?entry a core:Protein ;
         core:mnemonic ?mnemonic ;
         core:recommendedName|core:submittedName ?rname .
  ?rname core:fullName ?full_name .
  OPTIONAL{
    ?rname core:shortName ?short_name .
  }
  ?entry core:sequence [
    a core:Simple_Sequence ;
    rdf:value ?sequence ;
    core:mass ?mass
    ] .
  OPTIONAL{
    ?entry core:citation ?citation .
  }
  OPTIONAL {
    ?entry rdfs:seeAlso ?pdb .
    ?pdb core:database db:PDB .
  }
  BIND(STRLEN(?sequence) AS ?length)
  BIND(REPLACE(STR(?entry), uniprot:, "") AS ?id)
}

```

## `go`
```sparql
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX core: <http://purl.uniprot.org/core/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX oboinowl: <http://www.geneontology.org/formats/oboInOwl#>

SELECT DISTINCT ?entry ?category (GROUP_CONCAT(DISTINCT ?go_label, ",") AS ?go_labels)
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
FROM <http://rdf.integbio.jp/dataset/togosite/go>
WHERE {
  VALUES ?entry { uniprot:{{uniprot}} }
  OPTIONAL {
    GRAPH <http://rdf.integbio.jp/dataset/togosite/uniprot> {
      ?entry core:classifiedWith ?go .
    }
    GRAPH <http://rdf.integbio.jp/dataset/togosite/go> {
      ?go oboinowl:hasOBONamespace ?category ;
          rdfs:label ?go_label .
    }
  }
}
```

## `tissue`
```sparql
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX core: <http://purl.uniprot.org/core/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
SELECT DISTINCT ?entry
  (GROUP_CONCAT(DISTINCT ?tissue, ",") AS ?isolated_tissue) 
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot/tissues>
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE{
  VALUES ?entry { uniprot:{{uniprot}} }
  OPTIONAL {
    ?entry core:isolatedFrom/skos:prefLabel ?tissue .
  }
}

```

## `return`

```javascript
({ main, go, tissue }) => {
  let obj = main.results.bindings.map(data => {
    return Object.keys(data).reduce((obj, key) => {
      obj[key] = data[key].value;
      return obj;
    }, {});
  });
  if (obj[0]["short_name"]) obj[0]["full_name"]  += " (" + obj[0]["short_name"] + ")";
  if (obj[0]["length"]) obj[0]["length"] = obj[0]["length"].replace(/(\d)(?=(\d{3})+$)/g , '$1,');
  if (obj[0]["mass"]) obj[0]["mass"] = obj[0]["mass"].replace(/(\d)(?=(\d{3})+$)/g , '$1,');
  if (obj[0]["citation_number"]) obj[0]["citation_number"] = obj[0]["citation_number"].replace(/(\d)(?=(\d{3})+$)/g , '$1,');
  if (obj[0]["pdbs"]) obj[0]["pdbs"] = obj[0]["pdbs"].split(/,/).map(d => {
    return "<a href='" + d + "' target=\"_blank\">" + d.replace("http://rdf.wwpdb.org/pdb/", "") + "</a>";
  }).join(", ");
  obj[0]["biological_process"] = "";
  obj[0]["molecular_function"] = "";
  obj[0]["cellular_component"] = "";
  obj[0]["isolated_tissue"] = "";
  go.results.bindings.forEach((data) => {
    if (data.category) {
      obj[0][data.category.value] = makeList(data.go_labels.value.split(/,/));
    }
  });
  if (tissue.results.bindings[0].isolated_tissue.value)
    obj[0]["isolated_tissue"] = makeList(tissue.results.bindings[0].isolated_tissue.value.split(/,/));
  return obj;
  
  function makeList(strs) {
    return "<ul><li>" + strs.join("</li><li>") + "</li></ul>";
  }
};
```
