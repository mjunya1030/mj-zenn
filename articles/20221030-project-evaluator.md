---
title: "dbt Labs のベストプラクティス全部違反してみた。そして dbt project evaluator を使って全部直してみた。" # 記事のタイトル
emoji: "🛠️" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["dbt","analytics engineering","bigquery"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

## サマリ

dbt Labs では、dbt のプロジェクト、並びに変換パイプラインに関するベストプラクティスを紹介しています。
さらに、いくつかのベストプラクティスについては、自動で評価可能な dbt project evaluator というツールも公開されています。
今回は、dbt project evaluator で評価可能な、20 個のベストプラクティスを全て「違反」した dbt プロジェクトを１から作成し、このツールを当てて評価した上で、修正をかけました。
実際にツールがうまく検知してくれるのかを確認し、検知された項目を修正する場合の手続きとその難所や、現実的な運用方法をまとめています。
## dbt project evaluator について

dbt labs 謹製のパッケージです。

https://github.com/dbt-labs/dbt-project-evaluator

## dbt Labs のベストプラクティス

先に今回対象とする 20個の観点を列挙しておきます。
①いつどこでどんなテーブルをjoinするべきか等のモデリングの観点、②最低限記載するべき schema テストの観点、③最低限記載するべきメタ情報などドキュメンテーションの観点、④staging や reporting などの層を意識したディレクトリ構造の観点、⑤多段viewをどこまで許すかといったパフォーマンスの観点の合計5つの観点があるようです。

### モデリング
  - ソースへのダイレクトジョイン
  - ソースに依存する下流モデル
  - モデルのファンアウト
  - 複数ソースの結合
  - 上流コンセプトの再接合
  - ルートモデル
  - ソースのファンアウト
  - ダウンストリームモデルに依存するステージングモデル
  - 他のステージングモデルに依存するステージングモデル
  - 未使用のソース
### テスト
  - 主要キーの欠落テスト
  - テストカバレッジ
### ドキュメンテーション
  - ドキュメンテーションカバレッジ
  - 文書化されていないモデル
### 構造
  - モデルの命名規則
  - モデルディレクトリ
  - ソースディレクトリ
  - テストディレクトリ
### パフォーマンス
  - 連鎖したビューの依存性
  - エクスポージャ 親のマテリアライゼーション


## 事前準備

早速プロジェクトを作成しました。ベストプラクティスを読んだ上で、全部違反している project がこちらです。20個のベストプラクティスにどう違反してるかの対応も記載しています。


![](/images/project-evaluator/pipeline-before.png)

<参考リポジトリ>
https://github.com/mjunya1030/ga4-dbt-template


このパイプラインは、この zenn のアクセスログを解析している Google Analytics のログを BigQuery に蓄積したものをソースデータとし、デバイスやページごとにUU数やPV数を出すテーブルを生成する処理です。
やりたいことはシンプルですが、全て違反するように作ったため、かなり見辛いパイプラインになってると思います。


## dbt project evaluator の適用

違反しているパイプラインに dbt projeect evaluator を適用してみます。

```sh
$ poetry run dbt test --select package:dbt_project_evaluator

13:43:06  Running with dbt=1.3.1
13:43:06  Found 48 models, 36 tests, 0 snapshots, 0 analyses, 560 macros, 0 operations, 1 seed file, 3 sources, 1 exposure, 0 metrics
13:43:06  
13:43:10  Concurrency: 4 threads (target='target')
13:43:10  
13:43:10  1 of 20 START test dbt_utils_accepted_range_fct_chained_views_dependencies_distance__False___var_chained_views_threshold_  [RUN]
13:43:10  2 of 20 START test dbt_utils_accepted_range_fct_documentation_coverage_documentation_coverage_pct___var_documentation_coverage_target_  [RUN]
13:43:10  3 of 20 START test dbt_utils_accepted_range_fct_test_coverage_test_coverage_pct___var_test_coverage_target_  [RUN]
13:43:10  4 of 20 START test is_empty_fct_direct_join_to_source_ ......................... [RUN]
13:43:11  4 of 20 FAIL 2 is_empty_fct_direct_join_to_source_ ............................. [FAIL 2 in 1.08s]
13:43:11  5 of 20 START test is_empty_fct_exposure_parents_materializations_ ............. [RUN]
13:43:11  1 of 20 FAIL 1 dbt_utils_accepted_range_fct_chained_views_dependencies_distance__False___var_chained_views_threshold_  [FAIL 1 in 1.12s]
13:43:11  6 of 20 START test is_empty_fct_marts_or_intermediate_dependent_on_source_ ..... [RUN]
13:43:11  3 of 20 FAIL 1 dbt_utils_accepted_range_fct_test_coverage_test_coverage_pct___var_test_coverage_target_  [FAIL 1 in 1.49s]
13:43:11  7 of 20 START test is_empty_fct_missing_primary_key_tests_ ..................... [RUN]
13:43:11  2 of 20 FAIL 1 dbt_utils_accepted_range_fct_documentation_coverage_documentation_coverage_pct___var_documentation_coverage_target_  [FAIL 1 in 1.54s]
13:43:11  8 of 20 START test is_empty_fct_model_directories_ ............................. [RUN]
13:43:12  6 of 20 FAIL 1 is_empty_fct_marts_or_intermediate_dependent_on_source_ ......... [FAIL 1 in 0.82s]
13:43:12  9 of 20 START test is_empty_fct_model_fanout_ .................................. [RUN]
13:43:12  5 of 20 FAIL 1 is_empty_fct_exposure_parents_materializations_ ................. [FAIL 1 in 0.90s]
13:43:12  10 of 20 START test is_empty_fct_model_naming_conventions_ ..................... [RUN]
13:43:12  7 of 20 FAIL 7 is_empty_fct_missing_primary_key_tests_ ......................... [FAIL 7 in 0.73s]
13:43:12  11 of 20 START test is_empty_fct_multiple_sources_joined_ ...................... [RUN]
13:43:12  8 of 20 FAIL 1 is_empty_fct_model_directories_ ................................. [FAIL 1 in 0.97s]
13:43:12  12 of 20 START test is_empty_fct_rejoining_of_upstream_concepts_ ............... [RUN]
13:43:13  10 of 20 FAIL 1 is_empty_fct_model_naming_conventions_ ......................... [FAIL 1 in 0.65s]
13:43:13  13 of 20 START test is_empty_fct_root_models_ .................................. [RUN]
13:43:13  9 of 20 FAIL 1 is_empty_fct_model_fanout_ ...................................... [FAIL 1 in 0.77s]
13:43:13  14 of 20 START test is_empty_fct_source_directories_ ........................... [RUN]
13:43:13  11 of 20 FAIL 1 is_empty_fct_multiple_sources_joined_ .......................... [FAIL 1 in 0.65s]
13:43:13  15 of 20 START test is_empty_fct_source_fanout_ ................................ [RUN]
13:43:13  12 of 20 FAIL 1 is_empty_fct_rejoining_of_upstream_concepts_ ................... [FAIL 1 in 0.73s]
13:43:13  16 of 20 START test is_empty_fct_staging_dependent_on_marts_or_intermediate_ ... [RUN]
13:43:13  14 of 20 FAIL 3 is_empty_fct_source_directories_ ............................... [FAIL 3 in 0.66s]
13:43:13  17 of 20 START test is_empty_fct_staging_dependent_on_staging_ ................. [RUN]
13:43:13  13 of 20 FAIL 1 is_empty_fct_root_models_ ...................................... [FAIL 1 in 0.79s]
13:43:13  18 of 20 START test is_empty_fct_test_directories_ ............................. [RUN]
13:43:13  15 of 20 FAIL 2 is_empty_fct_source_fanout_ .................................... [FAIL 2 in 0.57s]
13:43:13  19 of 20 START test is_empty_fct_undocumented_models_ .......................... [RUN]
13:43:14  17 of 20 FAIL 1 is_empty_fct_staging_dependent_on_staging_ ..................... [FAIL 1 in 0.50s]
13:43:14  20 of 20 START test is_empty_fct_unused_sources_ ............................... [RUN]
13:43:14  16 of 20 FAIL 1 is_empty_fct_staging_dependent_on_marts_or_intermediate_ ....... [FAIL 1 in 0.64s]
13:43:14  18 of 20 FAIL 2 is_empty_fct_test_directories_ ................................. [FAIL 2 in 0.71s]
13:43:14  19 of 20 FAIL 7 is_empty_fct_undocumented_models_ .............................. [FAIL 7 in 0.77s]
13:43:14  20 of 20 FAIL 1 is_empty_fct_unused_sources_ ................................... [FAIL 1 in 0.49s]
13:43:14  
13:43:14  Finished running 20 tests in 0 hours 0 minutes and 7.99 seconds (7.99s).
13:43:14  
13:43:14  Completed with 20 errors and 0 warnings:
13:43:14  
13:43:14  Failure in test is_empty_fct_direct_join_to_source_ (models/marts/dag/dag.yml)
13:43:14    Got 2 results, configured to fail if != 0
13:43:14  
13:43:14    compiled Code at target/compiled/dbt_project_evaluator/models/marts/dag/dag.yml/is_empty_fct_direct_join_to_source_.sql
13:43:14  
13:43:14  Failure in test dbt_utils_accepted_range_fct_chained_views_dependencies_distance__False___var_chained_views_threshold_ (models/marts/performance/performance.yml)
13:43:14    Got 1 result, configured to fail if != 0
13:43:14  
13:43:14    compiled Code at target/compiled/dbt_project_evaluator/models/marts/performance/performance.yml/dbt_utils_accepted_range_fct_c_f4e4ed7d4720ea1b89a90d20835948ae.sql
13:43:14  
13:43:14  Failure in test dbt_utils_accepted_range_fct_test_coverage_test_coverage_pct___var_test_coverage_target_ (models/marts/tests/testing.yml)
13:43:14    Got 1 result, configured to fail if != 0
13:43:14  
13:43:14    compiled Code at target/compiled/dbt_project_evaluator/models/marts/tests/testing.yml/dbt_utils_accepted_range_fct_t_2fb17cd478c8bb66913b01eb1c42e454.sql
13:43:14  
13:43:14  Failure in test dbt_utils_accepted_range_fct_documentation_coverage_documentation_coverage_pct___var_documentation_coverage_target_ (models/marts/documentation/documentation.yml)
13:43:14    Got 1 result, configured to fail if != 0
13:43:14  
13:43:14    compiled Code at target/compiled/dbt_project_evaluator/models/marts/documentation/documentation.yml/dbt_utils_accepted_range_fct_d_60fa1e42bcaed7ea1fca15068fe7952f.sql
13:43:14  
13:43:14  Failure in test is_empty_fct_marts_or_intermediate_dependent_on_source_ (models/marts/dag/dag.yml)
13:43:14    Got 1 result, configured to fail if != 0
13:43:14  
13:43:14    compiled Code at target/compiled/dbt_project_evaluator/models/marts/dag/dag.yml/is_empty_fct_marts_or_intermediate_dependent_on_source_.sql
13:43:14  
13:43:14  Failure in test is_empty_fct_exposure_parents_materializations_ (models/marts/dag/dag.yml)
13:43:14    Got 1 result, configured to fail if != 0
13:43:14  
13:43:14    compiled Code at target/compiled/dbt_project_evaluator/models/marts/dag/dag.yml/is_empty_fct_exposure_parents_materializations_.sql
13:43:14  
13:43:14  Failure in test is_empty_fct_missing_primary_key_tests_ (models/marts/tests/testing.yml)
13:43:14    Got 7 results, configured to fail if != 0
13:43:14  
13:43:14    compiled Code at target/compiled/dbt_project_evaluator/models/marts/tests/testing.yml/is_empty_fct_missing_primary_key_tests_.sql
13:43:14  
13:43:14  Failure in test is_empty_fct_model_directories_ (models/marts/structure/structure.yml)
13:43:14    Got 1 result, configured to fail if != 0
13:43:14  
13:43:14    compiled Code at target/compiled/dbt_project_evaluator/models/marts/structure/structure.yml/is_empty_fct_model_directories_.sql
13:43:14  
13:43:14  Failure in test is_empty_fct_model_naming_conventions_ (models/marts/structure/structure.yml)
13:43:14    Got 1 result, configured to fail if != 0
13:43:14  
13:43:14    compiled Code at target/compiled/dbt_project_evaluator/models/marts/structure/structure.yml/is_empty_fct_model_naming_conventions_.sql
13:43:14  
13:43:14  Failure in test is_empty_fct_model_fanout_ (models/marts/dag/dag.yml)
13:43:14    Got 1 result, configured to fail if != 0
13:43:14  
13:43:14    compiled Code at target/compiled/dbt_project_evaluator/models/marts/dag/dag.yml/is_empty_fct_model_fanout_.sql
13:43:14  
13:43:14  Failure in test is_empty_fct_multiple_sources_joined_ (models/marts/dag/dag.yml)
13:43:14    Got 1 result, configured to fail if != 0
13:43:14  
13:43:14    compiled Code at target/compiled/dbt_project_evaluator/models/marts/dag/dag.yml/is_empty_fct_multiple_sources_joined_.sql
13:43:14  
13:43:14  Failure in test is_empty_fct_rejoining_of_upstream_concepts_ (models/marts/dag/dag.yml)
13:43:14    Got 1 result, configured to fail if != 0
13:43:14  
13:43:14    compiled Code at target/compiled/dbt_project_evaluator/models/marts/dag/dag.yml/is_empty_fct_rejoining_of_upstream_concepts_.sql
13:43:14  
13:43:14  Failure in test is_empty_fct_source_directories_ (models/marts/structure/structure.yml)
13:43:14    Got 3 results, configured to fail if != 0
13:43:14  
13:43:14    compiled Code at target/compiled/dbt_project_evaluator/models/marts/structure/structure.yml/is_empty_fct_source_directories_.sql
13:43:14  
13:43:14  Failure in test is_empty_fct_root_models_ (models/marts/dag/dag.yml)
13:43:14    Got 1 result, configured to fail if != 0
13:43:14  
13:43:14    compiled Code at target/compiled/dbt_project_evaluator/models/marts/dag/dag.yml/is_empty_fct_root_models_.sql
13:43:14  
13:43:14  Failure in test is_empty_fct_source_fanout_ (models/marts/dag/dag.yml)
13:43:14    Got 2 results, configured to fail if != 0
13:43:14  
13:43:14    compiled Code at target/compiled/dbt_project_evaluator/models/marts/dag/dag.yml/is_empty_fct_source_fanout_.sql
13:43:14  
13:43:14  Failure in test is_empty_fct_staging_dependent_on_staging_ (models/marts/dag/dag.yml)
13:43:14    Got 1 result, configured to fail if != 0
13:43:14  
13:43:14    compiled Code at target/compiled/dbt_project_evaluator/models/marts/dag/dag.yml/is_empty_fct_staging_dependent_on_staging_.sql
13:43:14  
13:43:14  Failure in test is_empty_fct_staging_dependent_on_marts_or_intermediate_ (models/marts/dag/dag.yml)
13:43:14    Got 1 result, configured to fail if != 0
13:43:14  
13:43:14    compiled Code at target/compiled/dbt_project_evaluator/models/marts/dag/dag.yml/is_empty_fct_staging_dependent_on_marts_or_intermediate_.sql
13:43:14  
13:43:14  Failure in test is_empty_fct_test_directories_ (models/marts/structure/structure.yml)
13:43:14    Got 2 results, configured to fail if != 0
13:43:14  
13:43:14    compiled Code at target/compiled/dbt_project_evaluator/models/marts/structure/structure.yml/is_empty_fct_test_directories_.sql
13:43:14  
13:43:14  Failure in test is_empty_fct_undocumented_models_ (models/marts/documentation/documentation.yml)
13:43:14    Got 7 results, configured to fail if != 0
13:43:14  
13:43:14    compiled Code at target/compiled/dbt_project_evaluator/models/marts/documentation/documentation.yml/is_empty_fct_undocumented_models_.sql
13:43:14  
13:43:14  Failure in test is_empty_fct_unused_sources_ (models/marts/dag/dag.yml)
13:43:14    Got 1 result, configured to fail if != 0
13:43:14  
13:43:14    compiled Code at target/compiled/dbt_project_evaluator/models/marts/dag/dag.yml/is_empty_fct_unused_sources_.sql
13:43:14  
13:43:14  Done. PASS=0 WARN=0 ERROR=20 SKIP=0 TOTAL=20
```

20個の観点についてチェックしているのですが、`Completed with 20 errors and 0 warnings:` と出ているので、漏れなく検知できているようです。
実行時間もこのテーブル数なら数秒で、CIに組み込むことも難しくは無さそうでした。
※dbt-project-evaluator の実行だけであれば、検査対象の project をビルドせず、クエリもしない。


## プロジェクトの修正

### 裏側の仕組み

プロジェクトの修正に入る前に、dbt project evaluator の仕組みと使い方を簡単に紹介します。

dbt project evaluator は、model の依存関係を解決し、親テーブルと子テーブルを関係を表現した DAG テーブルを作ります。

{{ dag テーブルのサンプル }}

このテーブルをベースとして、親・子テーブルの種類や数、多段 view の段数、ドキュメンテーションの状況などを解析し、違反している場合は、解析結果をまとめたテーブルを作ります。

最後に、解析結果をまとめたテーブルの行数が 0行以上でないかを dbt test でチェックしています。

つまり、dbt project evaluator で出てきた dbt test のコンパイルクエリを使えば、解析結果のテーブルにアクセスし、エラーとなっている箇所を特定できます。これを使って修正して行きます。

早速エラーを解決して行きます。

### Direct Join To Source

#### このエラーは何？

source で定義されたファイルが、 staging 層を介さずに、下流で join されている場合に発生します。

#### 何が悪いの？

Source 側にスキーマ変更があった場合、不用意に参照しているテーブルが多数あると、全て変更する必要が生じてしまいます。staging 層でラップすることで、source の変更に強くなります。

#### 修正

早速コンパイルされたコードを実行します。

```sql
    select *
    from `dbt_ga4_project_evaluator`.`fct_direct_join_to_source`
```

{{ 実行結果の画像 }}

parent = page_display_names とあるので、page_display_names が関連するテーブルと思われます。
※実行結果のテーブルの見方が少し難しいかもしれません。
dbt docs で確認すると、確かに子テーブルの一つである、rpt_access_count_by_page が内部で page_display_names を join してることがわかります。

page_display_names は staging 層にあらかじめ定義した stg_page_display_names で置き換えできそうなので、変更します。

{{ github のコミットの画像 }}


```
$ poetry run dbt build --select package:dbt_project_evaluator,+fct_direct_join_to_source


14:13:57  Found 48 models, 36 tests, 0 snapshots, 0 analyses, 560 macros, 0 operations, 1 seed file, 3 sources, 1 exposure, 0 metrics
14:13:57  
14:14:01  Concurrency: 4 threads (target='target')
14:14:01  
...
14:14:16  14 of 14 START test is_empty_fct_direct_join_to_source_ ........................ [RUN]
14:14:17  14 of 14 PASS is_empty_fct_direct_join_to_source_ .............................. [PASS in 0.73s]
14:14:17  
14:14:17  Finished running 7 view models, 6 table models, 1 test in 0 hours 0 minutes and 19.51 seconds (19.51s).
14:14:17  
14:14:17  Completed successfully
14:14:17  
```

無事 test が通りました。
ただし、変更前後でデータの値が変わっている可能性はあります。
ここは dbt build でデータが変わってないか等も確認する必要はあります。今回は合っているものとして次に進みます。


### Downstream Models Dependent on Source

#### このエラーは何？

source で定義されたファイルが、 staging 層を介さずに、下流で直接参照されている場合に発生します。

#### 何が悪いの？

sourceのテーブルはカラム名などは変わり得るため、sourceではなく、staging 層のテーブルを源流とする構成にしておくことで、モデリングの一貫性を保つことができます。
staging を介さずに参照されている source テーブルがあることは、一対一対応が崩れていることになります。

#### 修正

早速コンパイルされたコードを実行します。

```sql
select *
from `dbt_ga4_project_evaluator`.`fct_marts_or_intermediate_dependent_on_source`
```

{{ 実行結果の画像 }}

結果のテーブルは比較的わかりやすいです。
child = dim_pages とあるので dbt docs で確認すると、dim_pages が source の page_display_names を 直接参照 してることがわかります。

page_display_names は staging 層にあらかじめ定義した stg_page_display_names で置き換えできそうなので、変更します。

{{ github のコミットの画像 }}


```
$ poetry run dbt build --select package:dbt_project_evaluator,+fct_marts_or_intermediate_dependent_on_source
Configuration file exists at /Users/moritajunya/Library/Application Support/pypoetry, reusing this directory.

14:30:57  14 of 14 START test is_empty_fct_marts_or_intermediate_dependent_on_source_ .... [RUN]
14:30:58  14 of 14 PASS is_empty_fct_marts_or_intermediate_dependent_on_source_ .......... [PASS in 0.89s]
14:30:58  
14:30:58  Finished running 7 view models, 6 table models, 1 test in 0 hours 0 minutes and 18.48 seconds (18.48s).
14:30:58  
14:30:58  Completed successfully
14:30:58  
```

無事 test が通りました。


### Model Fanout

#### このエラーは何？

一つの model (source ではない) から、3つ以上の(末端)子テーブルが作られている場合に発生します。

#### 何が悪いの？

一つのテーブルのみを参照するテーブルが複数作られており、それらが最終的なレポートとしてBIツール等で利用されている場合、テーブルを作るのではなく、親となるテーブルを直接参照し、BIツール側で加工した方が、保守性が高く、効率的である可能性が高いです。
※これはBIツールに依存すると思います。Looker のようにクラウドのデータソースを都度参照するような場合は当てはまりますが、Tableauのようにデータソースをローカルで抽出する場合は、子テーブルとしてレポートを作っておくほうが性能面で有利な場合もあります。

#### 修正

早速コンパイルされたコードを実行します。

```sql
select *
from `dbt_ga4_project_evaluator`.`fct_model_fanout`
```

{{ 実行結果の画像 }}

結果のテーブルは比較的わかりやすいです。
parent = fct_events とあるので fct_events が関連するテーブルで、rpt_device_access_* と続くテーブルたちが不要であることが推察されます。
pv,uu,session について一つのテーブルにまとめた rpt_device_summary_with_uu がすでにあるので、これらは不要とし、削除してしまいます。

{{ github のコミットの画像 }}


```
$ poetry run dbt build --select package:dbt_project_evaluator,+fct_model_fanout
Configuration file exists at /Users/moritajunya/Library/Application Support/pypoetry, reusing this directory.

14:44:56  14 of 14 START test is_empty_fct_model_fanout_ ................................. [RUN]
14:44:57  14 of 14 PASS is_empty_fct_model_fanout_ ....................................... [PASS in 0.70s]
14:44:57  
14:44:57  Finished running 7 view models, 6 table models, 1 test in 0 hours 0 minutes and 16.42 seconds (16.42s).
14:44:57  
14:44:57  Completed successfully
14:44:57  
14:44:57  Done. PASS=14 WARN=0 ERROR=0 SKIP=0 TOTAL=14
```

無事 test が通りました。


### Multiple Sources Joined

#### このエラーは何？

複数の source を join したテーブルが staging 層にあるとき発生します。

#### 何が悪いの？

staging 層のテーブルはモデリングの源流となるので、可能な限り細かい粒度のものを持っておくことが理想です。
もし複数の source を join しているテーブルがあるなら、おそらくもっと細かい粒度が定義できることを意味するため、別のテーブルに分けた方が良い可能性があります。

#### 修正

早速コンパイルされたコードを実行します。

```sql
select *
from `dbt_ga4_project_evaluator`.`fct_multiple_sources_joined`
```

{{ 実行結果の画像 }}

結果のテーブルは比較的わかりやすいです。
child = stg_events_and_names で、source_parents = {analytics_272722196.events_*, analytics_272722196.page_display_names} とあるので、この2つを join してることが原因と思われます。
stg_events_and_names は定義しているが下流で使っていないので、削除してしまいます。

{{ github のコミットの画像 }}


```
$ poetry run dbt build --select package:dbt_project_evaluator,+fct_multiple_sources_joined
Configuration file exists at /Users/moritajunya/Library/Application Support/pypoetry, reusing this directory.
14:54:13  14 of 14 START test is_empty_fct_multiple_sources_joined_ ...................... [RUN]
14:54:14  14 of 14 PASS is_empty_fct_multiple_sources_joined_ ............................ [PASS in 0.82s]
14:54:14  
14:54:14  Finished running 7 view models, 6 table models, 1 test in 0 hours 0 minutes and 16.85 seconds (16.85s).
14:54:14  
14:54:14  Completed successfully
14:54:14  
14:54:14  Done. PASS=14 WARN=0 ERROR=0 SKIP=0 TOTAL=14
```

無事 test が通りました。
また、この修正によって、後述する Souce Fanout も解決されてしまいました。
こちらも source テーブルの扱いに関する観点なので、どちらかを治すと両方改善されることは起こりやすそうです。


### Rejoining of Upstream Concepts

#### このエラーは何？

子テーブルとして作ったテーブルが、同じ親をもつ別の子テーブルから参照されており、それ以外に使われていない時に発生します。

#### 何が悪いの？

これは不用意にモジュール化されてしまっていることを検知しています。
処理時間の節約にもならないため、一つのSQLに処理をまとめる方が良い可能性があります。
ただし、SQLの業種が増えるため、クエリが複雑すぎる場合は例外です。

#### 修正

早速コンパイルされたコードを実行します。

```sql
select *
from `dbt_ga4_project_evaluator`.`fct_rejoining_of_upstream_concepts`
```

{{ 実行結果の画像 }}

結果のテーブルは比較的わかりやすいです。
parent_and_child = rpt_device_summary となっているので、これが不要なモジュールのようです。。
親か子のどちらかのクエリに混ぜてしまおうと思います。

{{ github のコミットの画像 }}


```
$ poetry run dbt build --select package:dbt_project_evaluator,fct_rejoining_of_upstream_concepts

15:10:34  Found 43 models, 34 tests, 0 snapshots, 0 analyses, 560 macros, 0 operations, 1 seed file, 3 sources, 1 exposure, 0 metrics
15:10:34  
15:10:36  Concurrency: 4 threads (target='target')
15:10:36  
15:10:36  1 of 2 START sql table model dbt_ga4_project_evaluator.fct_rejoining_of_upstream_concepts  [RUN]
15:10:39  1 of 2 OK created sql table model dbt_ga4_project_evaluator.fct_rejoining_of_upstream_concepts  [CREATE TABLE (0.0 rows, 4.1 KB processed) in 3.07s]
15:10:39  2 of 2 START test is_empty_fct_rejoining_of_upstream_concepts_ ................. [RUN]
15:10:40  2 of 2 PASS is_empty_fct_rejoining_of_upstream_concepts_ ....................... [PASS in 1.03s]
15:10:40  
15:10:40  Finished running 1 table model, 1 test in 0 hours 0 minutes and 6.43 seconds (6.43s).
15:10:40  
15:10:40  Completed successfully
15:10:40  
15:10:40  Done. PASS=2 WARN=0 ERROR=0 SKIP=0 TOTAL=2
```

無事 test が通りました。
また、この修正によって、後述する Chained View Dependencies も解決されてしまいました。
こちらもパフォーマンス観点のプラクティスなので、どちらかを治すと両方改善されることは起こりやすそうです。


### Root Models

#### このエラーは何？

source を使っていないモデルがある場合に発生します。

#### 何が悪いの？

dbt でリネージが追えなくなってしまうため、一貫性を失いやすいです。
ただ、date_spineマクロなどで作られるカレンダーマスタのような source を必要としないテーブルもあります。これらは例外です。

#### 修正

早速コンパイルされたコードを実行します。

```sql
select *
from `dbt_ga4_project_evaluator`.`fct_root_models`
```

{{ 実行結果の画像 }}


child = access_count_by_device となっているので、これの参照テーブルを修正します。

{{ github のコミットの画像 }}


```
$ poetry run dbt build --select package:dbt_project_evaluator,fct_rejoining_of_upstream_concepts


15:17:08  Found 43 models, 34 tests, 0 snapshots, 0 analyses, 560 macros, 0 operations, 1 seed file, 3 sources, 1 exposure, 0 metrics
15:17:08  
15:17:12  Concurrency: 4 threads (target='target')
15:17:12  
15:17:12  1 of 2 START sql table model dbt_ga4_project_evaluator.fct_rejoining_of_upstream_concepts  [RUN]
15:17:15  1 of 2 OK created sql table model dbt_ga4_project_evaluator.fct_rejoining_of_upstream_concepts  [CREATE TABLE (0.0 rows, 4.1 KB processed) in 3.21s]
15:17:15  2 of 2 START test is_empty_fct_rejoining_of_upstream_concepts_ ................. [RUN]
15:17:16  2 of 2 PASS is_empty_fct_rejoining_of_upstream_concepts_ ....................... [PASS in 0.94s]
15:17:16  
15:17:16  Finished running 1 table model, 1 test in 0 hours 0 minutes and 7.77 seconds (7.77s).
15:17:16  
15:17:16  Completed successfully
15:17:16  
15:17:16  Done. PASS=2 WARN=0 ERROR=0 SKIP=0 TOTAL=2
```

無事 test が通りました。
しかし、この修正によって、前述の項で解決したはずの model fanout が再発してしまいました。
先にこちらの不具合から直した方が検討の手戻りがなかったかもしれません。
大量にエラーが出ている場合は、修正する観点の順番を考えておくか、何周かする作戦で挑む必要がありそうです。

今回は、rpt_device_summary と同じ情報を持ってると考え、access_count_by_device を削除して model fanout も解決します。
※ 後述する model naming conventions も、このテーブルを消したことで解決されます。



### Source Fanout

#### このエラーは何？

source から 3つ以上の staging 層のテーブルが生成されているときに発生します。

#### 何が悪いの？

複数のテーブルが source を参照していると、生データのフォーマットが変更された場合に、変更箇所が増えてしまいます。
そのため、staging のテーブルと source のテーブルは1:1に対応させて、変換の処理はシンプルになるようにしておくのが望ましいです。

#### 修正

※このエラーは前述の操作で解決したので、省略します。



### Staging Models Dependent on Downstream Models と Staging Models Dependent on Other Staging Models

#### このエラーは何？

staging 層なのに上流テーブルがあるときに発生します

#### 何が悪いの？

sourceではない、staging や別のテーブルを参照しているということは、mart、あるいは reporting 層のテーブルであることが示唆されます。

#### 修正

```
select *
from `dbt_ga4_project_evaluator`.`fct_staging_dependent_on_marts_or_intermediate`

union all

select *
from `dbt_ga4_project_evaluator`.`fct_staging_dependent_on_staging`

```

child = stg_access_count_by_date とあるので、このテーブルの命名を変更して対応します。

{{ コミットの画像 }}

```
$ poetry run dbt build --select package:dbt_project_evaluator,+fct_staging_dependent_on_staging +fct_staging_dependent_on_marts_or_intermediate


15:40:15  Found 42 models, 32 tests, 0 snapshots, 0 analyses, 560 macros, 0 operations, 1 seed file, 3 sources, 1 exposure, 0 metrics
15:40:15  
15:40:17  Concurrency: 4 threads (target='target')
15:40:17  
15:40:32  16 of 17 PASS is_empty_fct_staging_dependent_on_staging_ ....................... [PASS in 0.67s]
15:40:32  17 of 17 PASS is_empty_fct_staging_dependent_on_marts_or_intermediate_ ......... [PASS in 0.66s]
15:40:32  
15:40:32  Finished running 7 view models, 7 table models, 1 seed, 2 tests in 0 hours 0 minutes and 16.82 seconds (16.82s).
15:40:32  
15:40:32  Completed successfully
15:40:32  
15:40:32  Done. PASS=17 WARN=0 ERROR=0 SKIP=0 TOTAL=17
moritajunyanoMacBook-Pro:dbt-container-for-bigquery moritajunya$ 
```

### Unused Sources

#### このエラーは何？

使っていない Source がある時に発生します

#### 何が悪いの？

YMLで定義したけれどもモデルに取り込まれなかったソースか、廃止されたモデルで、YMLファイルのソースブロックの対応する行が同時に削除されなかったことを表します。これは、単にプロジェクト内にある必要のない残骸が蓄積されていることを表しています。

#### 修正

```
select *
from `dbt_ga4_project_evaluator`.`fct_unused_sources`
```

parent = article_types とあるので、これが使われていない source のようです。yamlファイルから消すことにします。

{{ コミットの画像 }}

```
$ poetry run dbt test --select package:dbt_project_evaluator,+fct_unused_sources

15:45:50  Found 42 models, 32 tests, 0 snapshots, 0 analyses, 560 macros, 0 operations, 1 seed file, 2 sources, 1 exposure, 0 metrics
15:45:50  
15:45:52  Concurrency: 4 threads (target='target')
15:45:52  
15:45:52  1 of 1 START test is_empty_fct_unused_sources_ ................................. [RUN]
15:45:53  1 of 1 PASS is_empty_fct_unused_sources_ ....................................... [PASS in 0.75s]
15:45:53  
15:45:53  Finished running 1 test in 0 hours 0 minutes and 2.67 seconds (2.67s).
15:45:53  
15:45:53  Completed successfully
15:45:53  
```


### Testing と Document

テストを記載することで、ドキュメントも充足するため、まとめて対応します。
この観点では、一意性テストと非 null テストが一番重要と思われます。
これはモデルの粒度を表すことになるので、粒度をチェックするテストがあれば、テーブルの重複等を検知でき、最低限のテストとして機能します。
カラムの組み合わせで一意になるような場合は、サロゲートキーを生成することでチェック可能になります。

#### 修正

```
select *
from `dbt_ga4_project_evaluator`.`fct_missing_primary_key_tests`
```

resource_name = fct_dim_events_with_pages,stg_events,rpt_access_count_by_date とあるので、これらにテストを記載しつつ、ドキュメントも拡充します。

{{ コミットの画像 }}

```
$ poetry run dbt test --select package:dbt_project_evaluator,+fct_missing_primary_key_tests +fct_test_coverage +fct_documentation_coverage
15:57:55  Found 42 models, 38 tests, 0 snapshots, 0 analyses, 560 macros, 0 operations, 1 seed file, 2 sources, 1 exposure, 0 metrics
15:57:55  
15:57:57  Concurrency: 4 threads (target='target')
15:57:57  
15:57:57  1 of 3 START test dbt_utils_accepted_range_fct_documentation_coverage_documentation_coverage_pct___var_documentation_coverage_target_  [RUN]
15:57:57  2 of 3 START test dbt_utils_accepted_range_fct_test_coverage_test_coverage_pct___var_test_coverage_target_  [RUN]
15:57:57  3 of 3 START test is_empty_fct_missing_primary_key_tests_ ...................... [RUN]
15:57:58  3 of 3 PASS is_empty_fct_missing_primary_key_tests_ ............................ [PASS in 0.94s]
15:57:58  1 of 3 PASS dbt_utils_accepted_range_fct_documentation_coverage_documentation_coverage_pct___var_documentation_coverage_target_  [PASS in 0.98s]
15:57:58  2 of 3 PASS dbt_utils_accepted_range_fct_test_coverage_test_coverage_pct___var_test_coverage_target_  [PASS in 1.20s]
15:57:58  
15:57:58  Finished running 3 tests in 0 hours 0 minutes and 2.77 seconds (2.77s).
15:57:58  
15:57:58  Completed successfully
15:57:58  
15:57:58  Done. PASS=3 WARN=0 ERROR=0 SKIP=0 TOTAL=3
```

不要なテーブルの削除等は Modeling の観点で実施していたため、testを記載するべき対象が少なく済みました。


### Structure

ディレクトリを修正します。


#### 修正

```
select *
from `dbt_ga4_project_evaluator`.`fct_source_directories`;

select *
from `dbt_ga4_project_evaluator`.`fct_test_directories`;

select *
from `dbt_ga4_project_evaluator`.`fct_model_directories`;

```

change_file_path_to とあるカラムに、適正と思われるファイルパスが記載されてます。
この通り修正して行きます。

{{ コミットの画像 }}

```
$ poetry run dbt test --select package:dbt_project_evaluator,+fct_missing_primary_key_tests +fct_test_coverage +fct_documentation_coverage
15:57:55  Found 42 models, 38 tests, 0 snapshots, 0 analyses, 560 macros, 0 operations, 1 seed file, 2 sources, 1 exposure, 0 metrics
15:57:55  
15:57:57  Concurrency: 4 threads (target='target')
15:57:57  
15:57:57  1 of 3 START test dbt_utils_accepted_range_fct_documentation_coverage_documentation_coverage_pct___var_documentation_coverage_target_  [RUN]
15:57:57  2 of 3 START test dbt_utils_accepted_range_fct_test_coverage_test_coverage_pct___var_test_coverage_target_  [RUN]
15:57:57  3 of 3 START test is_empty_fct_missing_primary_key_tests_ ...................... [RUN]
15:57:58  3 of 3 PASS is_empty_fct_missing_primary_key_tests_ ............................ [PASS in 0.94s]
15:57:58  1 of 3 PASS dbt_utils_accepted_range_fct_documentation_coverage_documentation_coverage_pct___var_documentation_coverage_target_  [PASS in 0.98s]
15:57:58  2 of 3 PASS dbt_utils_accepted_range_fct_test_coverage_test_coverage_pct___var_test_coverage_target_  [PASS in 1.20s]
15:57:58  
15:57:58  Finished running 3 tests in 0 hours 0 minutes and 2.77 seconds (2.77s).
15:57:58  
15:57:58  Completed successfully
15:57:58  
15:57:58  Done. PASS=3 WARN=0 ERROR=0 SKIP=0 TOTAL=3
```



### Model Naming Conventions

これはテーブル名が reporting や mart、 staging に合わせて、接頭辞をつけているかをチェックします。
デフォルトでは、  other_prefixes = stg_*, marts_prefixes = dim_* or fct_*, staging_prefixes = stg_* ,intermediate_prefixes = int_* のように指定されており、これに違反するとアラートが発生します。

参考: https://github.com/dbt-labs/dbt-project-evaluator#overriding-variables 

dbt_project.yml でこれらの値は  override できるので、接頭辞を変更したい場合は設定します。


#### 修正

modeling の修正によって対象となるテーブルが削除されたため、今回の修正は不要となりました。


### Directory Structure

これは ディレクトリが正しく命名されているか、また各モデルが適切なディレクトリの中にあるかをチェックするものです。
ディレクトリはデフォルトで、model_types = staging, intermediate, marts, other の4つが定義されていて、
それぞれ、staging_folder_name=staging,  intermediate_folder_name=intermediate, marts_folder_name=marts というディレクトリ名で指定されています。
これは変更することも増やすことも可能で、`<model_type>_folder_name` のような形で定義できます

```
# dbt_project.yml
# add an additional model type "util"

vars:
  dbt_project_evaluator:
    model_types: ['staging', 'intermediate', 'marts', 'other', 'util']
    util_folder_name: 'util'
    util_prefixes: ['util_']
```

今回はデフォルトの指定に沿ってディレクトリ、ファイル名を合わせることとします。


#### 修正

```
select *
from `dbt_ga4_project_evaluator`.`fct_source_directories`;

select *
from `dbt_ga4_project_evaluator`.`fct_test_directories`;

select *
from `dbt_ga4_project_evaluator`.`fct_model_directories`;

```

change_file_path_to とあるカラムに、適正と思われるファイルパスが記載されてます。
この通り修正して行きます。

{{ コミットの画像 }}

```
$ poetry run dbt test --select package:dbt_project_evaluator,+fct_missing_primary_key_tests +fct_test_coverage +fct_documentation_coverage
15:57:55  Found 42 models, 38 tests, 0 snapshots, 0 analyses, 560 macros, 0 operations, 1 seed file, 2 sources, 1 exposure, 0 metrics
15:57:55  
15:57:57  Concurrency: 4 threads (target='target')
15:57:57  
15:57:57  1 of 3 START test dbt_utils_accepted_range_fct_documentation_coverage_documentation_coverage_pct___var_documentation_coverage_target_  [RUN]
15:57:57  2 of 3 START test dbt_utils_accepted_range_fct_test_coverage_test_coverage_pct___var_test_coverage_target_  [RUN]
15:57:57  3 of 3 START test is_empty_fct_missing_primary_key_tests_ ...................... [RUN]
15:57:58  3 of 3 PASS is_empty_fct_missing_primary_key_tests_ ............................ [PASS in 0.94s]
15:57:58  1 of 3 PASS dbt_utils_accepted_range_fct_documentation_coverage_documentation_coverage_pct___var_documentation_coverage_target_  [PASS in 0.98s]
15:57:58  2 of 3 PASS dbt_utils_accepted_range_fct_test_coverage_test_coverage_pct___var_test_coverage_target_  [PASS in 1.20s]
15:57:58  
15:57:58  Finished running 3 tests in 0 hours 0 minutes and 2.77 seconds (2.77s).
15:57:58  
15:57:58  Completed successfully
15:57:58  
15:57:58  Done. PASS=3 WARN=0 ERROR=0 SKIP=0 TOTAL=3
```

### Chained View Dependencies

これは4段以上の多段viewとなっており、 materialization  を変えることで性能改善が見込めそうなテーブルがある時に発生します。
多段viewがあると、最後のviewを参照するクエリが重たくなる傾向にあります。途中のviewを table または incremental に変更することで、性能改善が期待できます。

#### 修正

modeling の修正によって対象となるテーブルが削除されたため、今回の修正は不要となりました。


### Exposure Parents Materializations

#### このエラーは何？

これは Exposure として定義されたモデルの materialization が view である場合に発生します。

#### 何が悪いの？

Exposure とは外部から利用で切るように定義されたモデルで、多数のアクセスが想定されます。
パフォーマンスの観点から table や incremental の materialization で提供するのが望ましいです。

#### 修正

```
select *
from `dbt_ga4_project_evaluator`.`fct_exposure_parents_materializations`
```

parent_model_name = fct_dim_events_with_pages とあるので、このテーブルの materialization を table に変更します。

{{ コミットの画像 }}

この修正によって、全ての test が解決するはずです。
改めてテストを実行してみます。

```
$ poetry run dbt test --select package:dbt_project_evaluator


16:54:04  Found 42 models, 38 tests, 0 snapshots, 0 analyses, 560 macros, 0 operations, 1 seed file, 2 sources, 1 exposure, 0 metrics
16:54:04  
16:54:07  Concurrency: 4 threads (target='target')
16:54:07  
16:54:07  1 of 20 START test dbt_utils_accepted_range_fct_chained_views_dependencies_distance__False___var_chained_views_threshold_  [RUN]
16:54:07  2 of 20 START test dbt_utils_accepted_range_fct_documentation_coverage_documentation_coverage_pct___var_documentation_coverage_target_  [RUN]
16:54:07  3 of 20 START test dbt_utils_accepted_range_fct_test_coverage_test_coverage_pct___var_test_coverage_target_  [RUN]
16:54:07  4 of 20 START test is_empty_fct_direct_join_to_source_ ......................... [RUN]
16:54:08  4 of 20 PASS is_empty_fct_direct_join_to_source_ ............................... [PASS in 0.98s]
16:54:08  5 of 20 START test is_empty_fct_exposure_parents_materializations_ ............. [RUN]
16:54:08  1 of 20 PASS dbt_utils_accepted_range_fct_chained_views_dependencies_distance__False___var_chained_views_threshold_  [PASS in 1.03s]
16:54:08  6 of 20 START test is_empty_fct_marts_or_intermediate_dependent_on_source_ ..... [RUN]
16:54:08  2 of 20 PASS dbt_utils_accepted_range_fct_documentation_coverage_documentation_coverage_pct___var_documentation_coverage_target_  [PASS in 1.30s]
16:54:08  7 of 20 START test is_empty_fct_missing_primary_key_tests_ ..................... [RUN]
16:54:08  3 of 20 PASS dbt_utils_accepted_range_fct_test_coverage_test_coverage_pct___var_test_coverage_target_  [PASS in 1.56s]
16:54:08  8 of 20 START test is_empty_fct_model_directories_ ............................. [RUN]
16:54:08  6 of 20 PASS is_empty_fct_marts_or_intermediate_dependent_on_source_ ........... [PASS in 0.63s]
16:54:08  9 of 20 START test is_empty_fct_model_fanout_ .................................. [RUN]
16:54:08  5 of 20 PASS is_empty_fct_exposure_parents_materializations_ ................... [PASS in 0.85s]
16:54:08  10 of 20 START test is_empty_fct_model_naming_conventions_ ..................... [RUN]
16:54:09  7 of 20 PASS is_empty_fct_missing_primary_key_tests_ ........................... [PASS in 0.67s]
16:54:09  11 of 20 START test is_empty_fct_multiple_sources_joined_ ...................... [RUN]
16:54:09  8 of 20 PASS is_empty_fct_model_directories_ ................................... [PASS in 0.72s]
16:54:09  12 of 20 START test is_empty_fct_rejoining_of_upstream_concepts_ ............... [RUN]
16:54:09  9 of 20 PASS is_empty_fct_model_fanout_ ........................................ [PASS in 0.64s]
16:54:09  13 of 20 START test is_empty_fct_root_models_ .................................. [RUN]
16:54:09  11 of 20 PASS is_empty_fct_multiple_sources_joined_ ............................ [PASS in 0.63s]
16:54:09  14 of 20 START test is_empty_fct_source_directories_ ........................... [RUN]
16:54:09  10 of 20 PASS is_empty_fct_model_naming_conventions_ ........................... [PASS in 0.80s]
16:54:09  15 of 20 START test is_empty_fct_source_fanout_ ................................ [RUN]
16:54:09  12 of 20 PASS is_empty_fct_rejoining_of_upstream_concepts_ ..................... [PASS in 0.58s]
16:54:09  16 of 20 START test is_empty_fct_staging_dependent_on_marts_or_intermediate_ ... [RUN]
16:54:10  13 of 20 PASS is_empty_fct_root_models_ ........................................ [PASS in 0.79s]
16:54:10  17 of 20 START test is_empty_fct_staging_dependent_on_staging_ ................. [RUN]
16:54:10  14 of 20 PASS is_empty_fct_source_directories_ ................................. [PASS in 0.69s]
16:54:10  18 of 20 START test is_empty_fct_test_directories_ ............................. [RUN]
16:54:10  15 of 20 PASS is_empty_fct_source_fanout_ ...................................... [PASS in 0.74s]
16:54:10  19 of 20 START test is_empty_fct_undocumented_models_ .......................... [RUN]
16:54:10  16 of 20 PASS is_empty_fct_staging_dependent_on_marts_or_intermediate_ ......... [PASS in 0.67s]
16:54:10  20 of 20 START test is_empty_fct_unused_sources_ ............................... [RUN]
16:54:10  17 of 20 PASS is_empty_fct_staging_dependent_on_staging_ ....................... [PASS in 0.68s]
16:54:11  18 of 20 PASS is_empty_fct_test_directories_ ................................... [PASS in 0.69s]
16:54:11  19 of 20 PASS is_empty_fct_undocumented_models_ ................................ [PASS in 0.76s]
16:54:11  20 of 20 PASS is_empty_fct_unused_sources_ ..................................... [PASS in 0.66s]
16:54:11  
16:54:11  Finished running 20 tests in 0 hours 0 minutes and 6.67 seconds (6.67s).
16:54:11  
16:54:11  Completed successfully
16:54:11  
16:54:11  Done. PASS=20 WARN=0 ERROR=0 SKIP=0 TOTAL=20
```

無事、20個全ての test が成功しています。


## 実際のプロジェクトに適用する際のポイント

### 適用すべきフェーズ
このツールのチェックはかなり厳しい印象で、テーブル数がそこそこある一般的なプロジェクトで適用したら、ほとんどがエラーになると思います。
というのも、ベストプラクティスとして守るべき観点と、ツールの設定に依存している観点が混在しているため、カスタマイズした状態で適用しないと、期待しないアラートが多数上がってしまいます。
初期構築の際を除いて、チェックする観点を取捨選択して適用することが前提になりそうです。
### 適用すべき観点
以下にあげる、staging に関連するテストは積極的にチェックするのが望ましいと思います。修正することで検知できる不具合が増え、また修正箇所も限定されるため効果的だと感じました。
- Direct Join To Source
- Downstream Models Dependent on Source
- Root Models
- Unused Sources
- Missing Primary Key Tests

一方で、modelの修正を余儀なくされてしまう観点については、修正範囲が広く、今あるクエリを修正するというよりは、新しく作ったクエリを汚さないための仕組みと捉えたほうが良いかもしれません。
- Model Fanout
- Sourve Fanout
- Rejoining of Upstream Concepts

また、ディレクトリ構成やテーブル命名については、設定に依存する部分が大きいため、適用する前に自分たちのプロジェクトに合わせてカスタムする必要があります。これも、修正するというよりは、project evaluator の設定を自分たちの構成に寄せるような使い方になると思います。
- Model Naming Conventions
- Model Directories
- Source Directories
- Test Directories

※特定の観点のみチェックする場合は、dbt test コマンドにセレクタをつけて実行します。dbt seed を使って、テーブルxテスト観点の単位で除外リストを作成することもできます。
### データセットの分割

project evaluator は、解析に使うテーブルを多数生成します。
素直に実行してしまうと、実行環境と同一のスキーマにテーブルを作成してしまいます。
これを防ぐため、パッケージ専用のスキーマを切り出します。
dbt_project.yml に設定します。

```
models:
  dbt_ga4:
      ga:
        # materialize all models in models/ga as tables
        +materialized: table
  dbt_project_evaluator: 
    + schema: project_evaluator
```

ただし、ルールの例外を入れるための seed テーブル(dbt_project_evaluator_exceptions)は同じスキーマに作られてしまうので、注意してください。

### 実行対象の指定

project evaluator を install すると、 dbt run を実行するたびに project evaluator が動いてしまいます。
これを避けるためには、project evaluator のパッケージを指定・除外するようにして実行します。

```
# 指定する
poetry run dbt build --select package:dbt_project_evaluator

# 除外する
poetry run dbt build --exclude package:dbt_project_evaluator

```
### run される対象

exclude する場合は、対象のモデルは build されません。
そのため、dbt で動作しないクエリもチェックできてしまいますし、動作しないことを検知できません。


### 実行環境の除外

project evaluator を実行したくない環境においては、dbt_project.yml で設定することができます

```
  dbt_project_evaluator: 
    + schema: project_evaluator
    +enabled: "{{ env_var('ENABLE_DBT_PROJECT_EVALUATOR', true) }}"

```

また、project evaluator によるエラーの種類も同様に設定できます。
```
tests:
  dbt_project_evaluator:
    +severity: "{{ env_var('DBT_PROJECT_EVALUATOR_SEVERITY', 'error') }}"
```


### その他

int_all_dag_relationships を見ることで、親子関係の全てのDAGを見ることができるテーブルがあります。
これを用いて、独自のチェック観点を作ることも可能です。
