# Protein attributes from Uniprot RDF（井手, 守屋）

## Parameters
* `uniprot`
  * default: P06493
  * example: P06493
  
## `uniprot_list`
```javascript
({ uniprot }) => {
  uniprot = uniprot.replace(/\s/g, "");
  if (uniprot) {
    return uniprot.split(",");
  } else {
    return false;
  }
};
```

## Endpoint
https://integbio.jp/togosite/sparql

## `main`
```sparql
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX core: <http://purl.uniprot.org/core/>
PREFIX db: <http://purl.uniprot.org/database/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT ?entry ?id ?mnemonic ?full_name ?short_name ?length ?mass 
  (GROUP_CONCAT(distinct ?pdb ; separator = ",") AS ?pdbs)
  (COUNT(?citation) AS ?citation_number)
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE{
  {{#if uniprot_list}}
  VALUES ?entry { {{#each uniprot_list}} uniprot:{{this}} {{/each}} }
  {{/if}}
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

## `go_bp`
```sparql
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX core: <http://purl.uniprot.org/core/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
SELECT DISTINCT ?entry
  (GROUP_CONCAT(DISTINCT ?go_bp_l, ",") AS ?biological_process)
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
FROM <http://rdf.integbio.jp/dataset/togosite/go>
WHERE{
  {{#if uniprot_list}}
  VALUES ?entry { {{#each uniprot_list}} uniprot:{{this}} {{/each}} }
  {{/if}}
  OPTIONAL{
    GRAPH <http://rdf.integbio.jp/dataset/togosite/uniprot> {
      ?entry core:classifiedWith ?go_bp .
    }
    GRAPH <http://rdf.integbio.jp/dataset/togosite/go> {
      ?go_bp rdfs:subClassOf* obo:GO_0008150 ;
             rdfs:label ?go_bp_l .
    }
  }
}

```

## `go_mf`
```sparql
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX core: <http://purl.uniprot.org/core/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
SELECT DISTINCT ?entry
  (GROUP_CONCAT(DISTINCT ?go_mf_l, ",") AS ?molecular_function)
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
FROM <http://rdf.integbio.jp/dataset/togosite/go>
WHERE{
  {{#if uniprot_list}}
  VALUES ?entry { {{#each uniprot_list}} uniprot:{{this}} {{/each}} }
  {{/if}}
  OPTIONAL{
    GRAPH <http://rdf.integbio.jp/dataset/togosite/uniprot> {
      ?entry core:classifiedWith ?go_mf .
    }
    GRAPH <http://rdf.integbio.jp/dataset/togosite/go> {
      ?go_mf rdfs:subClassOf* obo:GO_0003674 ;
             rdfs:label ?go_mf_l .
    }
  }
}
```

## `go_cc`
```sparql
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX core: <http://purl.uniprot.org/core/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
SELECT DISTINCT ?entry
  (GROUP_CONCAT(DISTINCT ?go_cc_l, ",") AS ?cellular_component)
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
FROM <http://rdf.integbio.jp/dataset/togosite/go>
WHERE{
  {{#if uniprot_list}}
  VALUES ?entry { {{#each uniprot_list}} uniprot:{{this}} {{/each}} }
  {{/if}}
  OPTIONAL{
    GRAPH <http://rdf.integbio.jp/dataset/togosite/uniprot> {
      ?entry core:classifiedWith ?go_cc .
    }
    GRAPH <http://rdf.integbio.jp/dataset/togosite/go> {
      ?go_cc rdfs:subClassOf* obo:GO_0005575 ;
             rdfs:label ?go_cc_l .
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
  {{#if uniprot_list}}
  VALUES ?entry { {{#each uniprot_list}} uniprot:{{this}} {{/each}} }
  {{/if}}
  OPTIONAL {
    ?entry core:isolatedFrom/skos:prefLabel ?tissue .
  }
}

```

## `return`

```javascript
({ main, go_bp, go_mf, go_cc, tissue }) => {
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
    return "<a href='" + d + "'>" + d.replace("http://rdf.wwpdb.org/pdb/", "") + "</a>";
  }).join(", ");
  obj[0]["biological_process"] = "";
  obj[0]["molecular_function"] = "";
  obj[0]["cellular_component"] = "";
  obj[0]["isolated_tissue"] = "";
  if (go_bp.results.bindings[0].biological_process.value)
    obj[0]["biological_process"] = "<ul><li>" + go_bp.results.bindings[0].biological_process.value.split(/,/).join("</li><li>") + "</li></ul>";
  if (go_mf.results.bindings[0].molecular_function.value)
    obj[0]["molecular_function"] = "<ul><li>" + go_mf.results.bindings[0].molecular_function.value.split(/,/).join("</li><li>") + "</li></ul>";
  if (go_cc.results.bindings[0].cellular_component.value)
    obj[0]["cellular_component"] = "<ul><li>" + go_cc.results.bindings[0].cellular_component.value.split(/,/).join("</li><li>") + "</li></ul>";
  if (tissue.results.bindings[0].isolated_tissue.value)
    obj[0]["isolated_tissue"] = "<ul><li>" + tissue.results.bindings[0].isolated_tissue.value.split(/,/).join("</li><li>") + "</li></ul>";
  return obj;
};
```