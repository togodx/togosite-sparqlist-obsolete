# 【作業保留中: 三橋】 事例10-3-1-1. MONDO_MedGen_PubMed
## MONDOのある疾患カテゴリの関連論文(PubMed ID)の数をサブカテゴリ別に表示する(MedGen利用)

* 入力：**疾患**
  * MONDOのある疾患カテゴリ。（デフォルトは全疾患 MONDO_0000001 (disease and disorder)が対象)
  * mondo_root変数で指定する（例：MONDO_0003847 (Mendelian diseaseの場合))
* 出力：**論文**数
   * 入力した疾患カテゴリに関連する**論文**数（疾患サブカテゴリ単位で集計）
* 備考
  * 疾患(MONDO)と論文(PubMed)の関係はMedGen RDFより取得
  * MONDOはrdf:subClassOfは多重継承のため、ひとつの遺伝子が複数の疾患カテゴリに含まれる場合はそれぞれのカテゴリでカウントされる

## Parameters

* `mondo_root`
  * default: MONDO_0000001 
  * example: MONDO_0000001 (disease and disorder), MONDO_0003847 (Mendelian disease), Search MONDO at https://monarchinitiative.org/
* `pubmed_flag`
  * default:
  * example: true (PUBMED_IDを表示する)、デフォルトはPubMED IDの個数を表示
 * `limit`
   * default: 100000
   * example:100000 (pubmed_flag=trueの時に表示される行数）
* `ep`
  * default: https://togovar-dev.biosciencedbc.jp/sparql 

## Endpoint

{{ ep }}

## `result`  

```sparql
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX mondo: <http://purl.obolibrary.org/obo/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX medgen: <http://med2rdf/ontology/medgen#>
PREFIX obo: <http://purl.obolibrary.org/obo/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

{{#unless pubmed_flag}}
SELECT ?mondo_first_level,  ?mondo_first_level_label, count(distinct ?mondo) AS ?num_mondo_ids
WHERE {
{{/unless}}
SELECT distinct ?mondo_first_tier As ?mondo_first_level,  STR(?mondo_first_tier_label) AS ?mondo_first_level_label, ?mondo, STR(?mondo_label) AS ?mondo_label, ?pubmed
WHERE {
  #   {{ mondo_root }}直下URI, {{ mondo_root }}直下ラベル, {{ mondo_root }}直下の子孫URI, {{ mondo_root }}直下の子孫ラベル
  {
    SELECT ?mondo_first_tier ?mondo_first_tier_label ?mondo ?mondo_label
    WHERE {
      # 第１階層直下のエントリリストを作成するサブクエリ
      {
        SELECT ?mondo_first_tier ?mondo_first_tier_label
        WHERE { 
          GRAPH <http://togovar.biosciencedbc.jp/mondo> { 
            ?mondo_first_tier rdfs:subClassOf <http://purl.obolibrary.org/obo/{{ mondo_root }}>.
            ?mondo_first_tier rdfs:label ?mondo_first_tier_label.
          }
        }
      }
  
      #  各第１階層直下のエントリ以下のエントリを再帰的に取得する。直下エントリ自身を含む
      GRAPH <http://togovar.biosciencedbc.jp/mondo>{
        ?mondo rdfs:subClassOf* ?mondo_first_tier.
        ?mondo rdfs:label ?mondo_label.
      }
   }
 }
  # MedGenが持つPubMed IDの対応表を利用して疾患とPubMedの関係を抽出する。
 GRAPH <http://togovar.biosciencedbc.jp/medgen>{
      ?mgconso_mondo rdfs:seeAlso ?mondo.
      ?medgen_cid medgen:mgconso ?mgconso_mondo.
      ?medgen_uid rdfs:seeAlso ?medgen_cid.
      ?medgen_uid dct:references ?pubmed.
      FILTER(STRSTARTS(STR(?pubmed), "http://rdf.ncbi.nlm.nih.gov/pubmed/"))
 } 
  # 論文件数が多いので2020/1以降に限定する。
 GRAPH <http://togovar.biosciencedbc.jp/pubmed>{
 #   ?pubmed  dct:issued ?issued_date.
#     FILTER(xsd:dateTime(?issued_date) > "2021-01"^^xsd:date)
 } 
{{#unless pubmed_flag}}
   }
 #} limit 100000   # 集計値は1,000,000論文に限定して母集団とする
} GROUP BY ?mondo_first_level ?mondo_first_level_label  ORDER BY DESC (?num_mondo_ids)
{{else}}
 } limit {{ limit }}
{{/unless}}
```