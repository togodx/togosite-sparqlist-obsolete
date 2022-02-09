# Gene attributes from PDB RDF（井手)

## Endpoint

https://integbio.jp/togosite/sparql

## Parameters
* `id`
  * default: 6TIW
  * example: 6TIW, 6W9K

## `main`

```sparql
PREFIX pdbr: <http://rdf.wwpdb.org/pdb/>
PREFIX pdbo: <http://rdf.wwpdb.org/schema/pdbx-v50.owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>

SELECT DISTINCT ?pdb ?id ?title ?method ?polypeptide_number ?pH ?keywords ?text ?species ?host
                (GROUP_CONCAT(DISTINCT ?nonpoly; separator="__") AS ?nonpolys)
                (GROUP_CONCAT(DISTINCT ?uniprot; separator="__") AS ?uniprots)
                (GROUP_CONCAT(DISTINCT ?uniprot_desc; separator="__") AS ?uniprot_descs)
WHERE {
  VALUES ?pdb { pdbr:{{id}} }
  ?pdb dc:title ?title ;
       pdbo:has_exptlCategory/pdbo:has_exptl/pdbo:exptl.method ?method .  
  OPTIONAL { ?pdb pdbo:has_exptl_crystal_growCategory/pdbo:has_exptl_crystal_grow/pdbo:exptl_crystal_grow.pH ?pH . }
  OPTIONAL { ?pdb pdbo:has_em_bufferCategory/pdbo:has_em_buffer/pdbo:em_buffer.pH ?pH . }

  ?pdb pdbo:has_struct_keywordsCategory/pdbo:has_struct_keywords/pdbo:struct_keywords.pdbx_keywords ?keywords .
  ?pdb pdbo:has_struct_keywordsCategory/pdbo:has_struct_keywords/pdbo:struct_keywords.text ?text .

  OPTIONAL {
    ?pdb pdbo:has_entity_src_genCategory/pdbo:has_entity_src_gen/pdbo:entity_src_gen.pdbx_gene_src_scientific_name ?species .
    ?pdb pdbo:has_entity_src_genCategory/pdbo:has_entity_src_gen/pdbo:entity_src_gen.pdbx_host_org_scientific_name ?host .
  }
  OPTIONAL {
    ?pdb pdbo:has_pdbx_entity_nonpolyCategory/pdbo:has_pdbx_entity_nonpoly/pdbo:pdbx_entity_nonpoly.name ?nonpoly .
  }
  {
    SELECT DISTINCT ?pdb COUNT(?polypeptide) AS ?polypeptide_number {
     ?pdb pdbo:has_entity_polyCategory/pdbo:has_entity_poly ?polypeptide.
     VALUES ?polypeptide_type { "polypeptide(L)" "polypeptide(D)"}
     ?polypeptide pdbo:entity_poly.type ?polypeptide_type .
     #?polypeptide rdfs:seeAlso ?uniprot_link . #DNAのentryを排除
    }
  }
  OPTIONAL {
    ?pdb pdbo:has_entityCategory/pdbo:has_entity ?entity_each .
    ?entity_each pdbo:referenced_by_struct_ref/pdbo:link_to_uniprot ?uniprot .
    ?entity_each pdbo:entity.pdbx_description ?uniprot_desc .
  }
  BIND(REPLACE(STR(?pdb), pdbr:, "") AS ?id)
}
```

## `return`

```javascript
({ main, id }) => {
  const data = main.results.bindings[0];
  const objs = main.results.bindings.map((elem) => ({
    ID: elem.id.value,
    url: elem.pdb.value,
    title: elem.title.value,
    method: elem.method.value,
    polypeptide_number: elem.polypeptide_number.value,
    pH: elem.pH?.value ?? "N/A",
    classification: elem.keywords.value,
    functional_keywords: elem.text.value,
    species: elem.species?.value ?? "",
    "host_(expression_system)": elem.host?.value ?? "",
    macromolecules: "",
    other_molecules: ""    
  }));

  objs[0].links = makeLink("https://www.ebi.ac.uk/pdbe/entry/pdb/"+id, "PDBe") + ", "
                  + makeLink("https://www.rcsb.org/structure/"+id, "RCSB") + ", "
                  + makeLink("https://pdbj.org/mine/summary/"+id, "PDBj")
  if (data.uniprots?.value)
    objs[0].macromolecules = makePairList(data.uniprots.value.split("__").map((url)=>url.replace("http://purl.uniprot.org/uniprot/", "")),
                                          data.uniprot_descs.value.split("__"),
                                          data.uniprots.value.split("__"));
  if (data.nonpolys?.value)
    objs[0].other_molecules = makeList(data.nonpolys.value.split("__"));
  return objs;

  function makeLink(url, text) {
    return "<a href=\"" + url + "\" target=\"_blank\">" + text + "</a>";
  }
  function makeList(strs) {
    return "<ul><li>" + strs.join("</li><li>") + "</li></ul>";
  }
  function makePairList(ids, labels, urls) {
    return makeList(ids.map((id, i)=>makeLink(urls[i], id) + " " + labels[i]));
  }
};
```