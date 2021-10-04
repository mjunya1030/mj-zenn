---
title: "tableauで時間の軸の見える散布図を作る" # 記事のタイトル
emoji: "📈" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["BigQuery","Tableau"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# ページ機能に履歴をつける

# モチベーション
## ビジネスの分析には時間的な変化が大事

## ビジネスの分析には相関が大事

## 時間的な変化と相関の両方を見たい

# 着眼点
## 散布図のプロットの動きを残像に残す

<div class='tableauPlaceholder' id='viz1633342713305' style='position: relative'><noscript><a href='#'><img alt='sample - 2016 ' src='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;sc&#47;scatter-plot-with-time-series&#47;sample&#47;1_rss.png' style='border: none' /></a></noscript><object class='tableauViz'  style='display:none;'><param name='host_url' value='https%3A%2F%2Fpublic.tableau.com%2F' /> <param name='embed_code_version' value='3' /> <param name='site_root' value='' /><param name='name' value='scatter-plot-with-time-series&#47;sample' /><param name='tabs' value='no' /><param name='toolbar' value='yes' /><param name='static_image' value='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;sc&#47;scatter-plot-with-time-series&#47;sample&#47;1.png' /> <param name='animate_transition' value='yes' /><param name='display_static_image' value='yes' /><param name='display_spinner' value='yes' /><param name='display_overlay' value='yes' /><param name='display_count' value='yes' /><param name='language' value='ja-JP' /></object></div>                <script type='text/javascript'>                    var divElement = document.getElementById('viz1633342713305');                    var vizElement = divElement.getElementsByTagName('object')[0];                    vizElement.style.width='100%';vizElement.style.height=(divElement.offsetWidth*0.75)+'px';                    var scriptElement = document.createElement('script');                    scriptElement.src = 'https://public.tableau.com/javascripts/api/viz_v1.js';                    vizElement.parentNode.insertBefore(scriptElement, vizElement);                </script>

# 解決策
## ページ機能と履歴の表示を組み合わせる
## 実装例

## 効能
- 椅子とテーブルのどちらも「昨年に比べて悪化」しているが、椅子の方が圧倒的に数も利益もオーダーが大きいので、より重点を置くべきとわかる
- 椅子はもっとも成長していたサブカテゴリなのにも関わらず、利益が悪化していることがわかる。
- 本棚は、唯一左下の方向に動いており、数量も利益も下がっている。
といったことが一度にわかる。


# まとめ
- 時系列のフィールドを `ページ` に置く
- `履歴の表示` をチェック
- `次の履歴を表示するマーク` で `すべて` を選択
- `表示` で `両方` を選択
- `マーク`の `サイズ` に時系列のフィールドを追加