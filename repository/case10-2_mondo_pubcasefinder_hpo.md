#  【作業保留中：三橋】 事例10-2. MONDO_PubCaseFinder_HPO
## MONDOのある疾患カテゴリの関連表現型を集計する

* 入力：**疾患**
  * MONDOのある疾患カテゴリ。（デフォルトは全疾患 MONDO_0000001 (disease and disorder)が対象)
  * mondo_root変数で指定する（例：MONDO_0003847 (Mendelian diseaseの場合))
* 出力：**表現型**数
   * 入力した疾患カテゴリに関連する**表現**数（サブカテゴリ単位で集計）
* 備考
  * 疾患(MONDO)と表現型(HPO)の関係はPubCaseFinder RDFより取得
  * MONDOはrdf:subClassOfは多重継承のため、ひとつの遺伝子が複数の疾患カテゴリに含まれる場合はそれぞれのカテゴリでカウントされる

## Parameters

* `mondo_root`
  * default: MONDO_0000001 
  * example: MONDO_0000001 (disease and disorder), MONDO_0003847 (Mendelian disease), Search MONDO at https://monarchinitiative.org/
* `hpo_flag`
  * default:
  * example: true (HPO_IDを表示する)、デフォルトはHPO IDの個数を表示
 * `limit`
   * default: 100000
   * example:100000 (hpo_flag=trueの時に表示される行数）
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
PREFIX oa: <http://www.w3.org/ns/oa#>
PREFIX medgen: <http://med2rdf/ontology/medgen#>
PREFIX sio: <http://semanticscience.org/resource/>
PREFIX ncit: <http://ncicb.nci.nih.gov/xml/owl/EVS/Thesaurus.owl#>

{{#unless hpo_flag}}
SELECT ?mondo_first_level,  ?mondo_first_level_label, count(?hpo) AS ?num_hpos
WHERE {
{{/unless}}
SELECT distinct ?mondo_first_tier As ?mondo_first_level,  STR(?mondo_first_tier_label) AS ?mondo_first_level_label, ?mondo, STR(?mondo_label) AS ?mondo_label, ?hpo
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

  # PubCaseFinderが持つMONDO->{OMIM, ORDO}->HPOの対応表を利用して疾患と表現型の関係を抽出する。
  GRAPH <https://pubcasefinder.dbcls.jp/rdf> {
      ?as sio:SIO_000628 [rdfs:seeAlso ?mondo];
          sio:SIO_000628 ?omim_or_ordo.
      ?an oa:hasTarget ?omim_or_ordo ;
          oa:hasBody ?hpo. 
  }
{{#unless hpo_flag}}
 } limit 1000000   # 集計値は1,000,000 HPO_IDsに限定して母集団とする
} GROUP BY ?mondo_first_level ?mondo_first_level_label  ORDER BY DESC (?num_hpos)
{{else}}
 } limit {{ limit }}
{{/unless}}
```