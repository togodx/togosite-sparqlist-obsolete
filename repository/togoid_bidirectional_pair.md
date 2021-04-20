# TogoID bidirectional pair

## Parameters

* `source` database
  * default: uniprot
* `target` database
  * default: hgnc
* `ids`: source identifiers
  * default: O15409,Q9BYF1

## `id_list`

```javascript
({ids}) => {
  list = ids.replace(/,/g, '  ').trim();
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
PREFIX drugbank: <https://identifiers.org/drugbank/>
PREFIX ec: <http://identifiers.org/ec-code/>
PREFIX ena: <http://identifiers.org/ena.embl/>
PREFIX ensembl_gene: <http://identifiers.org/ensembl/>
PREFIX ensembl_protein: <http://identifiers.org/ensembl/>
PREFIX ensembl_transcript: <http://identifiers.org/ensembl/>
PREFIX go: <http://purl.obolibrary.org/obo/GO_>
PREFIX hgnc: <http://identifiers.org/hgnc/>
PREFIX hgnc_symbol: <http://identifiers.org/hgnc.symbol/>
PREFIX hint: <http://purl.jp/10/hint/>
PREFIX hmdb: <https://identifiers.org/hmdb/HMDB>
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
PREFIX kegg_pathway: <https://identifiers.org/kegg.pathway/>
PREFIX kegg_reaction: <http://identifiers.org/kegg.reaction/>
PREFIX lrg: <http://identifiers.org/lrg/>
PREFIX mbgd_gene: <http://mbgd.genome.ad.jp/rdf/resource/gene/>
PREFIX mbgd_organism: <http://mbgd.genome.ad.jp/rdf/resource/organism/>
PREFIX meddra: <http://identifiers.org/meddra/>
PREFIX medgen: <http://identifiers.org/medgen/>
PREFIX mesh: <https://identifiers.org/mesh/>
PREFIX mgi: <http://identifiers.org/mgi/MGI:>
PREFIX mirbase: <http://identifiers.org/mirbase/>
PREFIX mondo: <http://purl.obolibrary.org/obo/MONDO_>
PREFIX nando: <http://nanbyodata.jp/ontology/nando#>
PREFIX ncbigene: <http://identifiers.org/ncbigene/>
PREFIX ncbiprotein: <http://identifiers.org/ncbiprotein/>
PREFIX oma_group: <http://identifiers.org/oma.grp/>
PREFIX oma_protein: <https://identifiers.org/oma.protein/>
PREFIX omim_gene: <http://identifiers.org/mim/>
PREFIX omim_phenotype: <https://identifiers.org/mim/>
PREFIX orphanet: <http://identifiers.org/orphanet.ordo/>
PREFIX pdb: <https://rdf.wwpdb.org/pdb/>
PREFIX pfam: <http://identifiers.org/pfam/>
PREFIX pubchem_compound: <https://identifiers.org/pubchem.compound/>
PREFIX pubchem_substance: <https://identifiers.org/pubchem.substance/>
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
  VALUES ?source_uri { {{#each id_list}} {{../source}}:{{this}} {{/each}} }
  {
    GRAPH <http://togoid.dbcls.jp/graph/{{source}}-{{target}}> {
      ?source_uri [] ?target_uri .
    }
  } UNION {
    GRAPH <http://togoid.dbcls.jp/graph/{{target}}-{{source}}> {
      ?target_uri [] ?source_uri .
    }
  }
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