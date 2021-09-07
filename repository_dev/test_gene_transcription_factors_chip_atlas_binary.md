# Target genes of TFs in ChIP-Atlas （大田・池田・小野・千葉）

## Description

- Data sources
    - ChIP-Atlas: [http://dbarchive.biosciencedbc.jp/kyushu-u/hg38/target/](http://dbarchive.biosciencedbc.jp/kyushu-u/hg38/target/)
    - The genes in `<Transcription Factor>.10.tsv` were defined to be target genes of the `<Transcription Factor>`.

- Query
    - Input
        - Ensembl gene ID of target genes
    - Output
        - Ensembl gene ID of TFs

## Endpoint

https://integbio.jp/togosite/sparql

## `minimal`

```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX ensembl: <http://identifiers.org/ensembl/>
PREFIX taxid: <http://identifiers.org/taxonomy/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?tf ?target
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/chip_atlas> {
    ?tf obo:RO_0002428 ?target .
  }
}
```

## `main`

```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX ensembl: <http://identifiers.org/ensembl/>
PREFIX taxid: <http://identifiers.org/taxonomy/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?parent ?parent_label ?child ?child_label
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/chip_atlas> {
    ?tf obo:RO_0002428 ?target .
  }
  GRAPH <http://rdf.integbio.jp/dataset/togosite/ensembl> {
    ?ebi_tf rdfs:label ?parent_label ;
            rdfs:seeAlso ?tf ;
            dc:identifier ?parent .
    ?ebi_target rdfs:label ?child_label ;
                rdfs:seeAlso ?target ;
                dc:identifier ?child .
  }
}
```

## `noTf`
```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX ensembl: <http://identifiers.org/ensembl/>
PREFIX taxid: <http://identifiers.org/taxonomy/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?parent ?parent_label ?child ?child_label
WHERE {
  GRAPH <http://rdf.integbio.jp/dataset/togosite/ensembl> {
    ?enst obo:SO_transcribed_from ?ebiensg .
    ?ebiensg obo:RO_0002162 taxid:9606 ; # in taxon
             faldo:location ?ensg_location ;
             rdfs:label ?child_label ;
             dc:identifier ?child .
    BIND (strbefore(strafter(str(?ensg_location), "GRCh38/"), ":") AS ?chromosome)
    FILTER (?chromosome IN ("1", "2", "3", "4", "5", "6", "7", "8", "9", "10",
                            "11", "12", "13", "14", "15", "16", "17", "18", "19", "20",
                            "21", "22", "X", "Y", "MT" ))
    BIND(URI(CONCAT("http://identifiers.org/ensembl/", ?child)) AS ?ensg)
  }
  FILTER NOT EXISTS {
    GRAPH <http://rdf.integbio.jp/dataset/togosite/chip_atlas> {
      ?tf obo:RO_0002428 ?ensg .
    }
  }
  BIND("none" AS ?parent)
  BIND("none" AS ?parent_label)
}
```

## `return`
```javascript
({main, noTf}) => {
  let tree = [{
    id: "root",
    label: "root node",
    root: true
  }];
  let chk = {};
  let data = main.results.bindings.concat(noTf.results.bindings);
  data.map(d => {
    if (!chk[d.parent.value]) {
      chk[d.parent.value] = true;
      tree.push({     
        id: d.parent.value,
        label: d.parent_label.value,
        leaf: false,
        parent: "root"
      })
    }
    tree.push({
      id: d.child.value,
      label: d.child_label.value,
      leaf: true,
      parent: d.parent.value
    })
  });
  
  return tree;
};
```
