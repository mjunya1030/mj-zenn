---
title: "GoogleAnalytics 4 のイベントを BigQuery で集計する" # 記事のタイトル
emoji: "📊" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["BigQuery","GoogleAnalytics"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## Google Analytics の データを細かく分析したい。
Google Analytics は GA4 というバージョンから BigQuery に無料でエクスポートできるようになりました。

しかし、GA4 のイベントデータは ARRAY や STRUCT で格納されているため、RDB の SQL に慣れている人だと困惑すると思います。

そこで、抽出するためのお役立ちクエリをまとめます。

## 課題感
GA4 のイベントは BigQuery にエクスポートすると以下のようなスキーマになります。

![](https://storage.googleapis.com/zenn-user-upload/rws941jt89v7oadhkh3wka5a7apd)

行=1 の中に、たくさんのデータが入っています。
これらは ARRAY 型となるため、単純な select 文でアクセスすると以下のエラーが出ます。

![](https://storage.googleapis.com/zenn-user-upload/euvf9xm6thr2d4dk574iau2p46sa)

しかも、イベントによって key:value の組み合わせが全然違います。

page_view の場合

![](https://storage.googleapis.com/zenn-user-upload/2u02v6umxzcb1cjsd1w2g88037gm)

これらのデータに Standard SQL からアクセスする場合は unnest を駆使する必要があります。

## 解決策
unnest を使ったクエリは以下のようになります。

```sql
select
  /** イベント基本情報 **/
  event_timestamp,
  event_name,

  /** ネストされた event_params を unnest して取得 **/
  ( /** イベントラベル **/
    select
      params.value.string_value
    from
      unnest(event_params) as params
    where
      params.key = "event_label"
  ) as event_label
from
  `{{ GCP_PROJECT_ID }}.analytics_hoge.events*`

```

## 解説
### unnest について
読んで字のごとく、ネストを解除する処理です。
BigQuery は1レコードにネストされた子要素のデータを持つことができます。

ネストされた子要素のデータにアクセスしたい場合は、子要素の数だけレコードを複製する必要があります。
この時使うのが unnest です。

![](https://storage.googleapis.com/zenn-user-upload/qvs5l0wmeihryx8hgw1teluu5y8j)

子要素のデータを別テーブルと見立て、cross join した場合に近い処理になります。
上図の通り、 unnest すると、ネストされてないレコードは重複するので注意が必要です。

### unnest を使ってGA4のデータを抽出する
GA4 のデータを解析する場合、unnest した上で、where 句で key を指定し、select句で value を指定すれば、狙ったイベントの特定の key を抽出できます。
例えば、 event_name = "this_is_event_name" の イベントラベル(key="event_label")」を抽出するクエリは以下のようになります。

```sql
select
  /** イベント基本情報 **/
  event_timestamp,
  event_name,
  params.value.string_value as event_label
from
  `{{ GCP_PROJECT_ID }}.analytics_hoge.events*`,
  unnest(event_params) as params
where
  params.key = "event_label"
  and
  event_name="{{ カスタムイベントのイベントアクション名など }}"
image.png
```

ところが、欲しいイベントパラメータの key はイベントラベルだけではないと思います。
key = page_location や key = ga_session_id なども取得したいと思うはずです。

しかし、where 句に key の数だけ or 条件で書いてしまうと、当然データは unnest しているため、パラメータの数だけ重複してしまいます。
with句を用いて、別テーブルで作成したのち、最後に join すれば…！と思うのですが、event_id のような便利な識別子はありません…。

### サブクエリで unnest を使う
BigQuery では、 select 句の中でも サブクエリ が使えます。
サブクエリ内で unnest してしまえば、joinし直す必要がありません。

https://cloud.google.com

```sql
select
  /** イベント基本情報系 **/
  user_pseudo_id,
  event_date,
  event_timestamp,
  event_name,

  /** ネストされた event_params を unnest して取得 **/
  ( /** イベントラベル **/
    select
      params.value.string_value
    from
      unnest(event_params) as params
    where
      params.key = "event_label"
  ) as event_label,
  ( /** ページURL **/
    select
      params.value.string_value
    from
      unnest(event_params) as params
    where
      params.key = "page_location"
  ) as page_location,
  ( /** セッションID **/
    select
      params.value.string_value
    from
      unnest(event_params) as params
    where
      params.key = "ga_session_id"
  ) as ga_session_id,
from
  `{{ GCP_PROJECT_ID }}.analytics_hoge.events*`
where
  event_name="page_view"
  or
  event_name="{{ カスタムイベントのイベントアクション名など }}"

from 句以後のクエリも短くなり、見通しも良くなります。
```

### まとめ
サブクエリ内で unnest することで、データの重複を避けて ARRAY型のGA4イベントデータを抽出しました。
Google のサービスでは、 Cloud logging をはじめとして ARRAY 型でエクスポートされるデータは多いため、 unnest を使う際はぜひご一考ください。