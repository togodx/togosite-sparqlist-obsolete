# TogoID alt 3: route (multi) を生成して、一つの SPARQL で join

## Parameters

* `source` database
  * default: uniprot
* `target` database
  * default: chebi
* `ids`
  * default: Q9D7Q1,Q690N0,Q9NYF8,Q4V339,A6NCE7,A7E2F4,P69849,A6NN73,Q92928,Q5T1J5,P0C7P4,Q6DN03,P09874,Q08211,Q5T4S7,P12270,Q9UPN3,P07814

## `routes`

```javascript
({source, target}) => {
  let config = {
    database: {
      variant: ["togovar"],
      gene: ["hgnc", "ncbigene", "ensembl_gene", "ensembl_transcript"],
      protein: ["uniprot", "chembl_target"],
      structure: ["pdb"],
      compound: ["pubchem_compound", "chembl_compound", "chebi"],
      nando: ["nando"],
      disease: ["mondo", "medgen", "omim_phenotype", "orphanet", "hp", "mesh"]
    },
    route: {
      variant: {
        variant: [[]],
        gene: [[]],
        protein: [["hgnc", "uniprot"]],
        structure: [["hgnc", "uniprot", "pdb"]],
        compound: [["hgnc", "uniprot", "reactome_reaction", "chebi"], ["hgnc", "uniprot", "chembl_target", "chembl_compound"]],
        nando: [["medgen", "mondo"]],
        disease: [["medgen"]]
      },
      gene: {
        gene: [[]],
        protein: [["uniprot"]],
        structure: [["uniprot", "pdb"]],
        compound: [["uniprot", "reactome_reaction", "chebi"], ["uniprot", "chembl_target", "chembl_compound"]],
        nando: [["ncbigene", "medgen", "mondo"]],
        disease: [["ncbigene", "medgen"]]
      },
      protein: {
        protein: [[]],
        structure: [[]],
        compound: [["reactome_reaction", "chebi"], ["chembl_target", "chembl_compound"]],
        nando: [["ncbigene", "medgen", "mondo"]],
        disease: [["ncbigene", "medgen"]]
      },
      structure: {
        structure: [[]],
        compound: [["uniprot", "reactome_reaction", "chebi"], ["uniprot", "chembl_target", "chembl_compound"]],
        nando: [["uniprot", "ncbigene", "medgen", "mondo"]],
        disease: [["uniprot", "ncbigene", "medgen"]]
      },
      compound: {
        compound: [[]],
        nando: [["chembl_compound", "mesh", "mondo", "medgen"]],
        disease: [["chembl_compound", "mesh", "mondo"]],
      },
      nando: {
        nando: [[]],
        disease: [["mondo"]]
      },
      disease: {
        disease: [["mondo"]]
      }
    }
  };
 
  let sourceSubject, targetSubject;
  for (let subject of Object.keys(config.database)) {
    if (config.database[subject].includes(source)) sourceSubject = subject;
    if (config.database[subject].includes(target)) targetSubject = subject;
  }
  
  // 例外処理
  // mondo - medgen - hp
  if (source == "mondo" && target == "hp") {
    config.route.disease.disease[0].push("medgen");
  } else if (target == "mondo" && source == "hp") {
     config.route.disease.disease[0].unshift("medgen");
  }
  // chembl_compound - mesh
  if ((sourceSubject == "compound" && target == "mesh") 
      || (targetSubject == "compound" && source == "mesh")) {
    config.route.compound.disease[0].pop();
  }
  
  let makeRoute = (source, target, route) => {
    route.unshift(source);
    route.push(target);
    return route.filter((x, i, self) => self.indexOf(x) === i);
  }
  
  let routes = [];
  for (let subject_1 of Object.keys(config.route)) {
    for (let subject_2 of Object.keys(config.route[subject_1])) {
      if (subject_1 == sourceSubject && subject_2 == targetSubject) {
        for (let route of config.route[subject_1][subject_2]) {
          routes.push(makeRoute(source, target, route));
        }
      } else if (subject_1 == targetSubject && subject_2 == sourceSubject) {
        for (let route of config.route[subject_1][subject_2]) {
          routes.push(makeRoute(source, target, route.reverse()));
        }
      }
    }
  }
  return routes;
}
```

## `pairListArray`
```javascript
({routes})=>{
  let array = [];
  for (let i = 0; i < routes.length; i++) {
    let route = routes[i];
    let list = [];
    for (let j = 0; j < route.length - 1; j++) {
      let id1 = "id_" + [j];
      let id2 = "id_" + [j + 1];
      if (j == 0) id1 = "source_uri";
      if (j == route.length - 2) id2 = "target_uri";
      list.push({source: route[j], target: route[j + 1], id1: id1, id2: id2});
    }
    let union = "UNION";
    if (i == routes.length - 1) union = "";
    array.push({list: list, union: union});
  }
  return array;
}
```

## `idList`

```javascript
({ids}) => {
  list = ids.replace(/,/g, ' ').trim();
  if (list.length > 0) {
    return list.split(/\s+/);
  }
  return false;
}
```

## Endpoint

http://ep6.dbcls.jp/togoid/sparql

## Query `result`

```sparql
PREFIX chebi: <http://identifiers.org/chebi/CHEBI:>
PREFIX chembl_compound: <http://identifiers.org/chembl.compound/>
PREFIX chembl_target: <http://identifiers.org/chembl.target/>
PREFIX ensembl_gene: <http://identifiers.org/ensembl/>
PREFIX ensembl_transcript: <http://identifiers.org/ensembl/>
PREFIX hgnc: <http://identifiers.org/hgnc/>
PREFIX hp: <http://purl.obolibrary.org/obo/HP_>
PREFIX medgen: <http://identifiers.org/medgen/>
PREFIX mesh: <http://identifiers.org/mesh/>
PREFIX mondo: <http://purl.obolibrary.org/obo/MONDO_>
PREFIX nando: <http://nanbyodata.jp/ontology/nando#>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>
PREFIX omim_phenotype: <http://identifiers.org/mim/>
PREFIX orphanet: <http://identifiers.org/orphanet.ordo/>
PREFIX pdb: <http://rdf.wwpdb.org/pdb/>
PREFIX pubchem_compound: <http://identifiers.org/pubchem.compound/>
PREFIX reactome_reaction: <http://identifiers.org/reactome/>
PREFIX togovar: <http://togovar.biosciencedbc.jp/variation/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>

SELECT DISTINCT ?source_id ?target_id #?source_uri ?target_uri
WHERE {
  VALUES ?source_uri { {{#each idList}} {{../source}}:{{this}} {{/each}} }
  {{#each pairListArray}}
  {
    SELECT DISTINCT ?source_uri ?target_uri
    WHERE {
    {{#each list}}
      {
        GRAPH <http://togoid.dbcls.jp/graph/{{source}}-{{target}}> {
          ?{{id1}} [] ?{{id2}} .
        }
      } UNION {
        GRAPH <http://togoid.dbcls.jp/graph/{{target}}-{{source}}> {
          ?{{id2}} [] ?{{id1}} .
        }
      }
    {{/each}}
    }
  } {{union}} 
  {{/each}}
  BIND (replace(str(?source_uri), {{source}}:, '') AS ?source_id)
  BIND (replace(str(?target_uri), {{target}}:, '') AS ?target_id)
}
```

## Return

```javascript
({result}) => {
  return result.results.bindings.map(data => {
    return Object.keys(data).reduce((obj, key) => {
      obj[key] = data[key].value;
      return obj;
    }, {});
  });
}
```
