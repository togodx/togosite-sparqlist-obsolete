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

## Parameters
* `categoryIds` (type: ensembl gene ID (TF))
  * example: ENSG00000275700,ENSG00000101544,ENSG00000048052
* `queryIds` (type: ensembl gene ID (target))
  * example: ENSG00000000005,ENSG00000002587,ENSG00000115942
* `mode` (type: string)
  * example: idList, objectList

## `input_genes`
```javascript
({ queryIds }) => {
  queryIds = queryIds.replace(/,/g, " ");
  if (queryIds.match(/[^\s]/)) {
    return queryIds.split(/\s+/);
  } else {
    return false;
  }
};
```

## `input_categories`
```javascript
({ categoryIds }) => {
  categoryIds = categoryIds.replace(/,/g, " ");
  if (categoryIds.match(/\S/)) {
    return categoryIds.split(/\s+/);
  }
};
```

## `main`

```sparql
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX ensembl: <http://identifiers.org/ensembl/>
PREFIX taxid: <http://identifiers.org/taxonomy/>
PREFIX faldo: <http://biohackathon.org/resource/faldo#>
PREFIX dc: <http://purl.org/dc/elements/1.1/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

{{#if mode}}
SELECT DISTINCT ?tf_id ?tf_label ?ensg
{{else}}
SELECT ?tf_id ?tf_label (COUNT(DISTINCT ?ensg) AS ?count)
{{/if}}
WHERE {
  {
    {{#if input_genes}}
    VALUES ?ensg { {{#each input_genes}} ensembl:{{this}} {{/each}} }
    {{/if}}
    GRAPH <http://rdf.integbio.jp/dataset/togosite/chip_atlas> {
      ?tf obo:RO_0002428 ?ensg .
    }
    GRAPH <http://rdf.integbio.jp/dataset/togosite/ensembl> {
      ?ebi_tf rdfs:label ?tf_label ;
              rdfs:seeAlso ?tf ;
              dc:identifier ?tf_id .
    }
  }
  UNION
  {
    {{#if input_genes}}
    VALUES ?ensg { {{#each input_genes}} ensembl:{{this}} {{/each}} }
    {{/if}}
    GRAPH <http://rdf.integbio.jp/dataset/togosite/ensembl> {
      ?enst obo:SO_transcribed_from ?ebiensg .
      ?ebiensg obo:RO_0002162 taxid:9606 ; # in taxon
               faldo:location ?ensg_location ;
               dc:identifier ?ensg_id .
      BIND (strbefore(strafter(str(?ensg_location), "GRCh38/"), ":") AS ?chromosome)
      FILTER (?chromosome IN ("1", "2", "3", "4", "5", "6", "7", "8", "9", "10",
                              "11", "12", "13", "14", "15", "16", "17", "18", "19", "20",
                              "21", "22", "X", "Y", "MT" ))
      BIND(URI(CONCAT("http://identifiers.org/ensembl/", ?ensg_id)) AS ?ensg)
    }
    FILTER NOT EXISTS {
      GRAPH <http://rdf.integbio.jp/dataset/togosite/chip_atlas> {
        ?tf obo:RO_0002428 ?ensg .
      }
    }
    BIND("none" AS ?tf_id)
    BIND("none" AS ?tf_label)
  }

  {{#if input_categories}}
  VALUES ?tf_id { {{#each input_categories}} "{{this}}" {{/each}} }
  {{/if}}
} ORDER BY ?tf_label
```

## `return`

```javascript
({ main, mode }) => {
  if (mode === "idList") {
    return Array.from(new Set(
      main.results.bindings.map((elem) =>
        elem.ensg.value.replace("http://identifiers.org/ensembl/", "")
      )
    ));
  } else if (mode === "objectList") {
    return main.results.bindings.map((elem) => ({
      id: elem.ensg.value.replace("http://identifiers.org/ensembl/", ""),
      attribute: {
        categoryId: elem.tf_id.value,
        label: elem.tf_label.value
      }
    }));
  } else {
    return main.results.bindings.map((elem) => ({
      categoryId: elem.tf_id.value,
      label: elem.tf_label.value,
      count: Number(elem.count.value),
    }));
  }
}
```