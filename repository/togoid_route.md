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
      chembl_compound: ["chembl_compound"],
      compound: ["pubchem_compound", "chebi"],
      nando: ["nando"],
      disease: ["mondo", "medgen", "omim_phenotype", "orphanet", "hp", "mesh"]
    },
    route: {
      variant: {
        variant: [],
        gene: [],
        protein: ["hgnc", "uniprot"],
        structure: ["hgnc", "uniprot", "pdb"],
        chembl_compound: ["hgnc", "uniprot", "chembl_target", "chembl_compound"],
        compound: ["hgnc", "uniprot", "reactome_reaction", "chebi"],
        nando: ["medgen", "mondo"],
        disease: ["medgen"]
      },
      gene: {
        gene: [],
        protein: ["uniprot"],
        structure: ["uniprot", "pdb"],
        chembl_compound: ["uniprot", "chembl_target", "chembl_compound"],
        compound: ["uniprot", "reactome_reaction", "chebi"],
        nando: ["ncbigene", "medgen", "mondo"],
        disease: ["ncbigene", "medgen"]
      },
      protein: {
        protein: [],
        structure: [],
        chembl_compound: ["chembl_target", "chembl_compound"],
        compound: ["reactome_reaction", "chebi"],
        nando: ["ncbigene", "medgen", "mondo"],
        disease: ["ncbigene", "medgen"]
      },
      structure: {
        structure: [],
        chembl_compound: ["uniprot", "chembl_target", "chembl_compound"],
        compound: ["uniprot", "reactome_reaction", "chebi"],
        nando: ["uniprot", "ncbigene", "medgen", "mondo"],
        disease: ["uniprot", "ncbigene", "medgen"]
      },
      chembl_compound: {
        chembl_compund: [],
        compound: [],
        nando: ["chembl_target", "uniprot", "ncbigene", "medgen", "mondo"],
        disease: ["chembl_target", "uniprot", "ncbigene", "medgen"]
      },
      compound: {
        comppund: [],
        nando: ["chebi", "reactome_reaction", "uniprot", "ncbigene", "medgen", "mondo"],
        disease: ["chebi", "reactome_reaction", "uniprot", "ncbigene", "medgen"]
      },
      nando: {
        nando: [],
        disease: ["mondo"]
      },
      disease: {
        disease: ["mondo"]
      }
    }
  };
 
  let sourceSubject, targetSubject;
  for (let subject of Object.keys(config.database)) {
    if (config.database[subject].includes(source)) sourceSubject = subject;
    if (config.database[subject].includes(target)) targetSubject = subject;
  }
  
  let makeRoute = (source, target, route) => {
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