# TogoID alt 3: route (multi) を生成して、一つの SPARQL で join

## Parameters

* `source` database
  * default: uniprot
  * example: ncbigene, ensembl_gene, uniprot, pdb, chebi, chembl_compound, pubchem_compound, glytoucan, mondo, mesh, nando, hp, togovar
* `target` database
  * default: chebi
## `routes`

```javascript
({source, target}) => {
  let config = {
    database: {
      glycan: ["glytoucan"],
      variant: ["togovar"],
      gene: ["hgnc", "ncbigene", "ensembl_gene", "ensembl_transcript"],
      protein: ["uniprot", "chembl_target"],
      structure: ["pdb"],
      compound: ["pubchem_compound", "chembl_compound", "chebi"],
      nando: ["nando"],
      hp: ["hp"],
      disease: ["mondo", "medgen", "omim_phenotype", "orphanet", "mesh"]
    },
    route: {
      glycan: {
        glycan: [[]],
        variant: [["uniprot", "hgnc"]],
        gene: [["uniprot"]],
        protein: [["uniprot"]],
        structure: [["uniprot", "pdb"]],
        compound: [["pubchem_compound"]],
        nando: [["doid", "mondo"]],
        hp: [["doid", "mondo", "medgen"]],
        disease: [["doid", "mondo"]]
      },
      variant: {
        variant: [[]],
        gene: [[]],
        protein: [["hgnc", "uniprot"]],
        structure: [["hgnc", "uniprot", "pdb"]],
        compound: [["hgnc", "uniprot", "reactome_reaction", "chebi"], ["hgnc", "uniprot", "chembl_target", "chembl_compound"]],
        nando: [["clinvar", "medgen", "mondo"]],
        hp: [["clinvar", "medgen"]],
        disease: [["clinvar", "medgen"]]
      },
      gene: {
        gene: [[]],
        protein: [["uniprot"]],
        structure: [["uniprot", "pdb"]],
        compound: [["uniprot", "reactome_reaction", "chebi"], ["uniprot", "chembl_target", "chembl_compound"]],
        nando: [["ncbigene", "medgen", "mondo"]],
        hp: [["ncbigene", "medgen"]],
        disease: [["ncbigene", "medgen"]]
      },
      protein: {
        protein: [[]],
        structure: [[]],
        compound: [["reactome_reaction", "chebi"], ["chembl_target", "chembl_compound"]],
        nando: [["ncbigene", "medgen", "mondo"]],
        hp: [["ncbigene", "medgen"]],
        disease: [["ncbigene", "medgen"]]
      },
      structure: {
        structure: [[]],
        compound: [["uniprot", "reactome_reaction", "chebi"], ["uniprot", "chembl_target", "chembl_compound"]],
        nando: [["uniprot", "ncbigene", "medgen", "mondo"]],
        hp: [["uniprot", "ncbigene", "medgen"]],
        disease: [["uniprot", "ncbigene", "medgen"]]
      },
      compound: {
        compound: [[]],
        nando: [["chembl_compound", "mesh", "mondo", "medgen"]],
        hp: [["chembl_compound", "mesh", "mondo", "medgen"]],
        disease: [["chembl_compound", "mesh", "mondo"]],
      },
      nando: {
        nando: [[]],
        hp: [["mondo", "medgen"]],
        disease: [["mondo"]]
      },
      hp: {
        hp: [[]],
        disease: [["medgen", "mondo"]]
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