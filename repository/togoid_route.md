# TogoID route

## Parameters

* `source` database
  * default: uniprot
* `target` database
  * default: chebi

## `route`

```javascript
({source, target}) => {
  let config = {
    database: {
      variant: ["togovar"],
      gene: ["hgnc", "ncbigene", "ensembl_gene", "ensembl_transcript"],
      protein: ["uniprot", "chembl_target"],
      structure: ["pdb"],
      compound: ["chembl_compound", "pubchem_compound", "chebi"],
      disease: ["mondo", "medgen", "omim_phenotype", "orphanet", "hp", "nando"]
    },
    route: {
      variant: {
        variant: [],
        gene: [],
        protein: ["hgnc", "uniprot"],
        structure: ["hgnc", "uniprot", "pdb"],
        compound: ["hgnc", "uniprot", "chembl_target", "chembl_compound"],
        disease: ["medgen"]
      },
      gene: {
        gene: [],
        protein: ["uniprot"],
        structure: ["uniprot", "pdb"],
        compound: ["uniprot", "chembl_target", "chembl_compound"],
        disease: ["ncbigene", "medgen"]
      },
      protein: {
        protein: [],
        structure: [],
        compound: ["chembl_target", "chembl_compound"],
      //  disease: ["omim_phenotype", "mondo"]
        disease: ["ncbigene", "medgen"]
      },
      structure: {
        structure: [],
        compound: ["uniprot", "chembl_target", "chembl_compound"],
      //  disease: ["uniprot", "omim_phenotype", "mondo"]
        disease: ["uniprot", "ncbigene", "medgen"]
      },
      compound: {
        comppund: [],
      //  disease: ["chembl_compound", "chembl_target", "uniprot", "omim_phenotype"]
        disease: ["chembl_compound", "chembl_target", "uniprot", "ncbigene", "medgen"]
      },
      disease: {
        disease: ["mondo"],
      }
    }
  };
 
  let sourceSubject, targetSubject;
  for (let subject of Object.keys(config.database)) {
    if (config.database[subject].includes(source)) sourceSubject = subject;
    if (config.database[subject].includes(target)) targetSubject = subject;
  }
  console.log( sourceSubject);
  
  let makeRoute = (source, target, route) => {
    if (source == "nando" && route[0] == "medgen") route.unshift("mondo");
    if (target == "nando" && route[route.length - 1] == "medgen") route.push("mondo");
    route.unshift(source);
    route.push(target);
    return route.filter((x, i, self) => self.indexOf(x) === i);
  }
  
  for (let subject_1 of Object.keys(config.route)) {
    for (let subject_2 of Object.keys(config.route[subject_1])) {
      if (subject_1 == sourceSubject && subject_2 == targetSubject) {
        return makeRoute(source, target, config.route[subject_1][subject_2]);
      }
      if (subject_1 == targetSubject && subject_2 == sourceSubject) {
        return makeRoute(source, target, config.route[subject_1][subject_2].reverse());
      }
    }
  }
}
```