---
title: "今までにKaggleコンペで使ったLB Probing手法について"
emoji: "🐈"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["Kaggle"]
published: true
published_at: 2022-12-03 07:00
---
## 概要

今までに私がKaggleコンペで使ったLB Probing手法(LB探索手法)についてまとめます。

## LB Probingとは何か？

- notebookコンペなどtestデータが隠蔽された条件において、LBボードのスコアのみを使ってテストデータに関する統計量を推定すること

## どのように役立つのか？

- テストデータに関する知見を他のチームより詳細に知ることができる
- 曖昧さに対して確実な裏打ちができる（特に質問に対して運営から回答がない（or遅い）場合に有効）

## LB Probingできる条件

LB Probingが可能な条件として以下があります。

1. 評価メトリクスの詳細が公開されている。あるいは確かな方法で推定できる
2. 不正解のラベルの割合を意図的にコントロールできる
3. LBスコアの精度が十分なこと

## 具体的にどうやるか？

### LB Probingの仕組み

手法自体はコンペによって異なりますが、基本的な仕組みはほとんど同じです。

以下の3つの情報を使ってtestデータに関する統計量を推定します。

1. 普通に提出したLBスコア
2. 不正解の割合を意図的に変化させた（通常増やす）LBスコア
3. LBスコアの算出アルゴリズム

### LB Probingの例

例として、BirdCLEFF 2022で使ったtestデータを使って計算できる任意の統計量を推定する方法は以下です。

1. ある程度正解のラベルの割合が高いモデルを作って提出する
2. テストデータを使って統計量pを計算する（pは0-1に正規化する）
3. 1のモデルの正の予測値のうち、一定の割合pで正負を反転して提出する
4. 1, 2のLBスコアとメトリクスの算出方法からpの値を逆算する

## 今までに使ったLB Probing手法の目的別まとめ

| 目的                                   | 推定した統計量                       | コンペ            | 手法                                                                                                                                        |
| -------------------------------------- | ------------------------------------ | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| より詳細な評価メトリクスを知る         | precision / recall                   | GBR[^1]           | https://www.kaggle.com/competitions/tensorflow-great-barrier-reef/discussion/302130                                                         |
|                                        | サブグループごとの部分スコア         | BirdCLEF 2022[^2] | https://www.kaggle.com/competitions/birdclef-2022/discussion/322419                                                                         |
|                                        | モデルの正例の予測比率               | BirdCLEF 2022     | https://www.kaggle.com/competitions/birdclef-2022/discussion/322606                                                                         |
| test dataにおけるラベルの分布を知る    | frameあたりのGT labelの割合          | GBR               | https://www.kaggle.com/competitions/tensorflow-great-barrier-reef/discussion/302156                                                         |
| 評価メトリクスのアルゴリズムを推定する | -                                    | BirdCLEF 2022     | 仮説から予測されるLBの推定値が実際のLBの観測値と一致することを確認する。https://www.kaggle.com/competitions/birdclef-2022/discussion/321883 |
| テストデータ中のリークの割合を推定する | テストデータ中の訓練データとの重複率 | 4sq[^3]           | https://www.kaggle.com/competitions/foursquare-location-matching/discussion/336047                                                          |

[^1]: https://www.kaggle.com/competitions/tensorflow-great-barrier-reef
[^2]: https://www.kaggle.com/competitions/birdclef-2022
[^3]: https://www.kaggle.com/competitions/foursquare-location-matching
