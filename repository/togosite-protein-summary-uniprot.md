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
PREFIX keywords: <http://purl.uniprot.org/keywords/>
PREFIX db: <http://purl.uniprot.org/database/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
SELECT DISTINCT ?entry ?id ?mnemonic ?full_name ?short_name ?length ?mass 
(GROUP_CONCAT(distinct ?pdb ; separator = ",") AS ?pdbs)
  (COUNT(?citation) AS ?citation_number)
  (GROUP_CONCAT(DISTINCT ?kw_mf_l, ",") AS ?molecular_function)
  (GROUP_CONCAT(DISTINCT ?kw_cc_l, ",") AS ?cellular_component)
  (GROUP_CONCAT(DISTINCT ?kw_bp_l, ",") AS ?biological_process)
  (GROUP_CONCAT(DISTINCT ?tissue, ",") AS ?isolated_tissue) 
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot/keywords>
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot/tissues>
WHERE{
  {{#if uniprot_list}}
  VALUES ?entry { {{#each uniprot_list}} uniprot:{{this}} {{/each}} }
  {{/if}}
  ?entry a core:Protein ;
         core:mnemonic ?mnemonic ;
         core:recommendedName|core:submittedName ?rname .
  ?rname core:fullName ?full_name .
  OPTIONAL{
    ?rname core:shortName ?sname .
  }
  ?entry core:sequence [
    a core:Simple_Sequence ;
    rdf:value ?sequence ;
    core:mass ?mass
    ] .
  OPTIONAL{
    ?entry core:citation ?citation .
  }
  OPTIONAL{ 
    ?entry core:classifiedWith ?kw_mf .
    ?kw_mf a core:Concept ;
           rdfs:subClassOf+ keywords:9992 ;
           skos:prefLabel ?kw_mf_l .
  }
  OPTIONAL{ 
    ?entry core:classifiedWith ?kw_cc . 
    ?kw_cc a core:Concept ;
           rdfs:subClassOf+ keywords:9998 ;
           skos:prefLabel ?kw_cc_l .
  }
  OPTIONAL{ 
    ?entry core:classifiedWith ?kw_bp .
    ?kw_bp a core:Concept ;
           rdfs:subClassOf+ keywords:9999 ;
           skos:prefLabel ?kw_bp_l .
  }
  OPTIONAL {
    ?entry core:isolatedFrom/skos:prefLabel ?tissue .
  }
  OPTIONAL {
    ?entry rdfs:seeAlso ?pdb .
    ?pdb core:database db:PDB .
  }
  BIND(CONCAT("", ?sname) AS ?shortname)
  BIND(IF(STRLEN(?shortname)=0,"-", ?shortname) AS ?short_name)  
  BIND(STRLEN(?sequence) AS ?length)
  BIND(REPLACE(STR(?entry), uniprot:, "") AS ?id)
}

```

## `return`

```javascript
({ main }) => {
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
  if (obj[0]["biological_process"]) obj[0]["biological_process"] = "<ul><li>" + obj[0]["biological_process"].split(/,/).join("</li><li>") + "</li></ul>";
  if (obj[0]["molecular_function"]) obj[0]["molecular_function"] = "<ul><li>" + obj[0]["molecular_function"].split(/,/).join("</li><li>") + "</li></ul>";
  if (obj[0]["cellular_component"]) obj[0]["cellular_component"] = "<ul><li>" + obj[0]["cellular_component"].split(/,/).join("</li><li>") + "</li></ul>";
  if (obj[0]["isolated_tissue"]) obj[0]["isolated_tissue"] = "<ul><li>" + obj[0]["isolated_tissue"].split(/,/).join("</li><li>") + "</li></ul>";
  if (obj[0]["pdbs"]) obj[0]["pdbs"] = obj[0]["pdbs"].split(/,/).map(d => {
    return "<a href='" + d + "'>" + d.replace("http://rdf.wwpdb.org/pdb/", "") + "</a>";
  }).join(", ");
  return obj;
};
```
