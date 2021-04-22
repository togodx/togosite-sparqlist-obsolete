# TogoID alt 3: route を生成して、一つの SPARQL で join

## Parameters

* `source` database
  * default: uniprot
* `target` database
  * default: chebi
* `ids`
  * default: Q9D7Q1,Q690N0,Q9NYF8,Q4V339,A6NCE7,A7E2F4,P69849,A6NN73,Q92928,Q5T1J5,P0C7P4,Q6DN03,P09874,Q08211,Q5T4S7,P12270,Q9UPN3,P07814

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
      disease: ["mondo", "medgen", "omim_phenotype", "orphanet", "hp", "nando", "mesh"]
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
  console.log(sourceSubject);
  
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

## `pairList`
```javascript
({route})=>{
  let list = [];
  for (let i = 0; i < route.length - 1; i++) {
    let id1 = "id_" + [i];
    let id2 = "id_" + [i + 1];
    if (i == 0) id1 = "source_uri";
    if (i == route.length - 2) id2 = "target_uri";
    list.push({source: route[i], target: route[i + 1], id1: id1, id2: id2});
  }
  return list;
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
PREFIX affy_probeset: <http://identifiers.org/affy.probeset/>
PREFIX bioproject: <http://identifiers.org/bioproject/>
PREFIX biosample: <http://identifiers.org/biosample/>
PREFIX ccds: <http://identifiers.org/ccds/>
PREFIX chebi: <http://identifiers.org/chebi/CHEBI:>
PREFIX chembl_compound: <http://identifiers.org/chembl.compound/>
PREFIX chembl_target: <http://identifiers.org/chembl.target/>
PREFIX civic_gene: <http://civic.genome.wustl.edu/links/genes/>
PREFIX clinvar: <http://ncbi.nlm.nih.gov/clinvar/variation/>
PREFIX dbsnp: <http://identifiers.org/dbsnp/>
PREFIX dgidb: <FIXME>
PREFIX doid: <http://purl.obolibrary.org/obo/DOID_>
PREFIX drugbank: <http://identifiers.org/drugbank/>
PREFIX ec: <http://identifiers.org/ec-code/>
PREFIX ena: <http://identifiers.org/ena.embl/>
PREFIX ensembl_gene: <http://identifiers.org/ensembl/>
PREFIX ensembl_protein: <http://identifiers.org/ensembl/>
PREFIX ensembl_transcript: <http://identifiers.org/ensembl/>
PREFIX go: <http://purl.obolibrary.org/obo/GO_>
PREFIX hgnc: <http://identifiers.org/hgnc/>
PREFIX hgnc_symbol: <http://identifiers.org/hgnc.symbol/>
PREFIX hint: <http://purl.jp/10/hint/>
PREFIX hmdb: <http://identifiers.org/hmdb/HMDB>
PREFIX homologene: <http://identifiers.org/homologene/>
PREFIX hp: <http://purl.obolibrary.org/obo/HP_>
PREFIX human_protein_atlas: <http://www.proteinatlas.org/>
PREFIX insdc: <http://identifiers.org/insdc/>
PREFIX instruct: <http://purl.jp/10/instruct/>
PREFIX intact: <http://identifiers.org/intact/>
PREFIX interpro: <http://identifiers.org/interpro/>
PREFIX kegg_compound: <http://identifiers.org/kegg.compound/>
PREFIX kegg_disease: <http://identifiers.org/kegg.disease/>
PREFIX kegg_drug: <http://identifiers.org/kegg.drug/>
PREFIX kegg_genes: <http://identifiers.org/kegg.genes/>
PREFIX kegg_orthology: <http://identifiers.org/kegg.orthology/>
PREFIX kegg_pathway: <http://identifiers.org/kegg.pathway/>
PREFIX kegg_reaction: <http://identifiers.org/kegg.reaction/>
PREFIX lrg: <http://identifiers.org/lrg/>
PREFIX mbgd_gene: <http://mbgd.genome.ad.jp/rdf/resource/gene/>
PREFIX mbgd_organism: <http://mbgd.genome.ad.jp/rdf/resource/organism/>
PREFIX meddra: <http://identifiers.org/meddra/>
PREFIX medgen: <http://identifiers.org/medgen/>
PREFIX mesh: <http://identifiers.org/mesh/>
PREFIX mgi: <http://identifiers.org/mgi/MGI:>
PREFIX mirbase: <http://identifiers.org/mirbase/>
PREFIX mondo: <http://purl.obolibrary.org/obo/MONDO_>
PREFIX nando: <http://nanbyodata.jp/ontology/nando#>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>
PREFIX ncbiprotein: <http://identifiers.org/ncbiprotein/>
PREFIX oma_group: <http://identifiers.org/oma.grp/>
PREFIX oma_protein: <http://identifiers.org/oma.protein/>
PREFIX omim_gene: <http://identifiers.org/mim/>
PREFIX omim_phenotype: <http://identifiers.org/mim/>
PREFIX orphanet: <http://identifiers.org/orphanet.ordo/>
PREFIX pdb: <http://rdf.wwpdb.org/pdb/>
PREFIX pfam: <http://identifiers.org/pfam/>
PREFIX pubchem_compound: <http://identifiers.org/pubchem.compound/>
PREFIX pubchem_substance: <http://identifiers.org/pubchem.substance/>
PREFIX pubmed: <http://identifiers.org/pubmed/>
PREFIX reactome_pathway: <http://identifiers.org/reactome/>
PREFIX reactome_reaction: <http://identifiers.org/reactome/>
PREFIX refseq_genomic: <http://identifiers.org/refseq/>
PREFIX refseq_protein: <http://identifiers.org/refseq/>
PREFIX refseq_rna: <http://identifiers.org/refseq/>
PREFIX rgd: <http://identifiers.org/rgd/>
PREFIX rhea: <http://identifiers.org/rhea/>
PREFIX sra_accession: <http://identifiers.org/insdc.sra/>
PREFIX sra_analysis: <http://identifiers.org/insdc.sra/>
PREFIX sra_experiment: <http://identifiers.org/insdc.sra/>
PREFIX sra_project: <http://identifiers.org/insdc.sra/>
PREFIX sra_run: <http://identifiers.org/insdc.sra/>
PREFIX sra_sample: <http://identifiers.org/insdc.sra/>
PREFIX taxonomy: <http://identifiers.org/taxonomy/>
PREFIX togovar: <http://togovar.biosciencedbc.jp/variation/>
PREFIX uniprot: <http://purl.uniprot.org/uniprot/>
PREFIX wikipathways: <http://identifiers.org/wikipathways/>

SELECT DISTINCT ?source_id ?target_id ?source_uri ?target_uri
WHERE {
  VALUES ?source_uri { {{#each idList}} {{../source}}:{{this}} {{/each}} }
  {{#each pairList}}
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