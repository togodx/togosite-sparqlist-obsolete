# Interaction summary for TogoSite report（守屋）

- reaction (reactome), PPI (IntAct), ChEMBL assay を取得してまとめてテーブルに

## Parameters
* `id`
  * default: P12270
  * example: P12270, 15996
* `type`
  * default: uniprot
  * example: uniprot, chebi
  
## `type_chk`
```javascript
({ type }) => {
  let obj = {};
  obj[type] = true;
  return obj;
};
```

## `typeObj`
```javascript
async ({ type, id }) => {
  if (type == "uniprot") return false;
  const togoidApi = "https://integbio.jp/togosite/sparqlist/api/togoid_route_sparql";
  const fetchReq = async (url, body) => {
    let options = {
      method: 'POST',
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'application/x-www-form-urlencoded'
      }
    }
    if (body) options.body = body;
    return await fetch(url, options).then(res=>res.json());
  }
  let pair = await fetchReq(togoidApi, "source=chebi&target=chembl_compound&ids=" + id);
  return pair.map(d => d.target_id);
}
```

## Endpoint
https://integbio.jp/togosite/sparql

## `reaction`
```sparql
PREFIX biopax: <http://www.biopax.org/release/biopax-level3.owl#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT DISTINCT ?reaction ?reaction_label ?uniprot ?uniprot_name ?uniprot_uri ?chebi ?chebi_name ?chebi_uri
FROM <http://rdf.integbio.jp/dataset/togosite/reactome>
FROM <http://rdf.integbio.jp/dataset/togosite/chebi>
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
  VALUES ?reaction_type { biopax:BiochemicalReaction biopax:TemplateReaction biopax:Degradation }
  VALUES ?has_component { biopax:left biopax:right biopax:product }
  {{#if type_chk.uniprot}}
  VALUES ?uniprot { "{{id}}"^^xsd:string }
  {{/if}}
  {{#if type_chk.chebi}}
    VALUES ?chebi { "CHEBI:{{id}}"^^xsd:string }
  {{/if}}
  ?react a ?reaction_type ;
         biopax:displayName ?reaction_label ;
         biopax:xref [
           biopax:db "Reactome"^^xsd:string ;
           biopax:id ?reaction ;
         ] ;
         ?has_component ?complex .
  ?complex biopax:component*/biopax:entityReference/biopax:xref [
    biopax:db "UniProt"^^xsd:string ;
    biopax:id ?uniprot
  ] .
  ?complex biopax:component*/biopax:entityReference/biopax:xref [
    biopax:db "ChEBI"^^xsd:string ;
    biopax:id ?chebi
  ] .
  BIND(IRI(CONCAT ("http://purl.uniprot.org/uniprot/", ?uniprot)) AS ?uniprot_uri)
  BIND(IRI(CONCAT ("http://purl.obolibrary.org/obo/", REPLACE(?chebi, ":", "_"))) AS ?chebi_uri)
  GRAPH <http://rdf.integbio.jp/dataset/togosite/uniprot> {
    ?uniprot_uri up:recommendedName/up:fullName ?uniprot_name .
  }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/chebi> {
    ?chebi_uri rdfs:label ?chebi_name .
  }
}
```

## `ppi`
```sparql
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
SELECT DISTINCT ?uniprot_uri ?type ?target_uri ?target_name ?intact
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
  {{#if type_chk.uniprot}}
  VALUES ?type { up:Non_Self_Interaction up:Self_Interaction }
  VALUES ?uniprot_uri { uniprot:{{id}} }
  ?uniprot_uri a up:Protein ;
           up:interaction [ a ?type ;
                            ^up:interaction ?target_uri ;
                            up:participant ?intact
                          ] .
  ?target_uri up:recommendedName/up:fullName ?target_name .
  {{/if}}
}
```

## `assay`
```sparql
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX taxon: <http://identifiers.org/taxonomy/>
PREFIX cco: <http://rdf.ebi.ac.uk/terms/chembl#>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX up: <http://purl.uniprot.org/core/>
PREFIX chembl: <http://rdf.ebi.ac.uk/resource/chembl/molecule/>
SELECT DISTINCT ?chembl_uri ?chembl_name ?assay_type ?uniprot_uri {{#if chembl}}?uniprot_name{{/if}}
FROM <http://rdf.integbio.jp/dataset/togosite/chembl>
FROM <http://rdf.integbio.jp/dataset/togosite/uniprot>
WHERE {
  {{#if type_chk.uniprot}}
  VALUES ?uniprot_uri { uniprot:{{id}} }
  ?chembl_uri a cco:SmallMolecule ;
          skos:prefLabel ?chembl_name ;
          cco:hasActivity/cco:hasAssay ?assay .
  ?assay cco:hasTarget/skos:exactMatch/skos:exactMatch ?uniprot_uri .
  ?assay cco:assayType ?assay_type .
  ?uniprot_uri a cco:UniprotRef .
  {{/if}}
  {{#if chembl}}
  VALUES ?chembl_uri { {{#each chembl}} chembl:{{this}} {{/each}} }
  ?chembl_uri a cco:SmallMolecule ;
          skos:prefLabel ?chembl_name ;
          cco:hasActivity/cco:hasAssay ?assay .
  ?assay cco:hasTarget/skos:exactMatch/skos:exactMatch ?uniprot_uri .
  ?assay cco:assayType ?assay_type .
  ?uniprot_uri a cco:UniprotRef .
  GRAPH <http://rdf.integbio.jp/dataset/togosite/chebi> {
    ?uniprot_uri up:recommendedName/up:fullName ?uniprot_name .
  }
  {{/if}}
}
```

## `return`
```javascript
({type, reaction, ppi, assay})=>{
  let res = [];
  if (type == "chebi") {
    for (let d of reaction.results.bindings) {
      res.push({
        "interaction": "Reaction",
        "details": d.reaction_label.value,
        "details_link": "http://identifiers.org/reactome/" + d.reaction.value,
        "target": "<a href='" + d.uniprot_uri.value + "'>" + d.uniprot.value + "</a> " + d.uniprot_name.value
      })
	}
    for (let d of assay.results.bindings) {
      res.push({
        "interaction": "ChEMBL assay",
        "details": d.chembl_uri.value.replace("http://rdf.ebi.ac.uk/resource/chembl/molecule/", "") + " Assay type: " + d.assay_type.value,
        "target": "<a href='" + d.uniprot_uri.value + "'>" + d.uniprot_uri.value.replace("http://purl.uniprot.org/uniprot/", "") + "</a> " + d.uniprot_name.value
      })
    }
    return res;
  }
  for (let d of reaction.results.bindings) {
    res.push({
      "interaction": "Reaction (Reactome)",
      "details": d.reaction_label.value,
      "details_link": "http://identifiers.org/reactome/" + d.reaction.value,
      "target": "<a href='" + d.chebi_uri.value + "'>" + d.chebi.value + "</a> " + d.chebi_name.value
    })
  }
  for (let d of ppi.results.bindings) {
    if (d.type.value.match(/Non_Self_Interaction/) && d.uniprot_uri.value == d.target_uri.value) continue;
    res.push({
      "interaction": "Protein-protein interaction (IntAct)",
      "details": d.intact.value.replace("http://purl.uniprot.org/intact/", ""),
      "details_link": d.intact.value,
      "target": "<a href='" + d.target_uri.value + "'>" + d.target_uri.value.replace("http://purl.uniprot.org/uniprot/", "") + "</a> " + d.target_name.value
    })
  }  
  for (let d of assay.results.bindings) {
    res.push({
      "interaction": "ChEMBL assay",
      "details": "Assay type: " + d.assay_type.value,
      "target": "<a href='" + d.chembl_uri.value + "'>" + d.chembl_uri.value.replace("http://rdf.ebi.ac.uk/resource/chembl/molecule/", "") + "</a> " + d.chembl_name.value
    })
  }
  return res;
}

```