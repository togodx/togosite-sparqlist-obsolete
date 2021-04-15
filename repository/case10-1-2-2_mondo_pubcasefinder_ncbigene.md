# 【作業保留中：三橋】事例10-1-2-2. MONDO_PubCaseFinder_NCBIGene
## MONDOのある疾患カテゴリの関連遺伝子(NCBI Gene)の数をサブカテゴリ別に表示する(PubCaseFinder利用)

* 入力：**疾患**
  * MONDOのある疾患カテゴリ。（デフォルトは全疾患 MONDO_0000001 (disease and disorder)が対象)
  * mondo_root変数で指定する（例：MONDO_0003847 (Mendelian diseaseの場合))
* 出力：**遺伝子**数
   * 入力した疾患カテゴリに関連する**遺伝子**数（サブカテゴリ単位で集計）
* 備考
  * 疾患(MONDO)と遺伝子(NCBI Gene)の関係はPubCaseFinder RDFより取得
  * MONDOはrdf:subClassOfは多重継承のため、ひとつの遺伝子が複数の疾患カテゴリに含まれる場合はそれぞれのカテゴリでカウントされる
  * 入力と出力を反対にしたクエリが事例10-4.

## Parameters

* `mondo_root`
  * default: MONDO_0000001 
  * example: MONDO_0000001 (disease and disorder), MONDO_0003847 (Mendelian disease), Search MONDO at https://monarchinitiative.org/
* `gene_flag`
  * default:
  * example: true (NCBI_GENE_IDを表示)、デフォルトは遺伝子数を表示
* `limit`
   * default: 100
   * example: 100 (gene_flag=trueの時に表示される行数）
* `ep`
  * default: https://integbio.jp/rdf/sparql 

## Endpoint

{{ ep }}

## `result`  

```sparql
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX mondo: <http://purl.obolibrary.org/obo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX oboInOwl: <http://www.geneontology.org/formats/oboInOwl#>
PREFIX medgen: <http://med2rdf/ontology/medgen#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX ncit: <http://ncicb.nci.nih.gov/xml/owl/EVS/Thesaurus.owl#>

{{#if gene_flag}}
SELECT ?mondo_first_tier As ?mondo_first_level,  STR(?mondo_first_tier_label) AS ?mondo_first_level_label, ?mondo, STR(?mondo_label) AS ?mondo_label, ?ncbi_gene
{{else}}
SELECT str(?mondo_first_tier_label) As ?mondo_category count(distinct ?ncbi_gene) As ?num_genes
{{/if}}
WHERE {
  #   {{ mondo_root }}直下URI, {{ mondo_root }}直下ラベル, {{ mondo_root }}直下の子孫URI, {{ mondo_root }}直下の子孫ラベル
  {
    SELECT ?mondo_first_tier ?mondo_first_tier_label ?mondo ?mondo_label
    WHERE {
      # 第１階層直下のエントリリストを作成するサブクエリ
      {
        SELECT ?mondo_first_tier ?mondo_first_tier_label
        WHERE { 
          GRAPH <http://integbio.jp/rdf/ontology/mondo> { 
            ?mondo_first_tier rdfs:subClassOf <http://purl.obolibrary.org/obo/{{ mondo_root }}>.
            ?mondo_first_tier rdfs:label ?mondo_first_tier_label.
          }
        }
      }
  
      #  各第１階層直下のエントリ以下のエントリを再帰的に取得する。直下エントリ自身を含む
      GRAPH <http://integbio.jp/rdf/ontology/mondo>{
        ?mondo rdfs:subClassOf* ?mondo_first_tier.
        ?mondo rdfs:label ?mondo_label.
      }
   }
 }
  # PubCaseFinderが持つMONDOとNCBI Geneの対応表を利用して疾患と遺伝子の関係を抽出する。
  GRAPH <https://pubcasefinder.dbcls.jp/rdf> {
      ?as sio:SIO_000628 [rdfs:seeAlso ?mondo] ;
              sio:SIO_000628 ?ncbi_gene .
      ?ncbi_gene rdf:type ncit:C16612 .
  }
{{#if gene_flag}}
{{#if limit}}
} limit {{ limit }}
{{else}}
}
 {{/if}}
{{else}}
} GROUP BY ?mondo_first_tier_label  ORDER BY DESC (?num_genes)
{{/if}}
```