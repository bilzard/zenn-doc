---
title: "今までにKaggleコンペで使ったLB Probing手法について"
emoji: "🐈"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["Kaggle"]
published: true
published_at: 2022-12-03 07:00
---
## 概要

今までに私がKaggleコンペで使ったLB Probing手法(LB探索手法)についてまとめます。個々の手法の詳細については各コンペのディスカッションで既に共有済みなのでここでは説明しません。ここでは、LB Probingにあまり馴染みがない人に対して、LB Probingでどんなことができるか、具体的にどうやってるのかについてざっくりとしたイメージを持ってもらうことを目的とします。

## LB Probingとは何か？

- notebookコンペなどtestデータが隠蔽された条件において、LBボードのスコアのみを使ってテストデータに関する統計量を推定すること

## どのように役立つのか？

- テストデータに関する知見を他のチームより詳細に知ることができる
- コンペ設定の曖昧さに対して確実な裏打ちができる（特に質問に対して運営から回答がない（or遅い）場合に有効）

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

### LB Probingの具体例

具体的な計算を示した方がわかりやすいと思うので、以下では次のような設定のコンペについてLB Probingの例を示します。

* 二値クラス判別タスク
* テストデータの件数はN=100
* 評価メトリクスはaccuracy

あなたの開発したモデルはaccuracy=0.7のLBが得られているとします。ここでLBから「モデルの予測した100件中70件は正解である」ことがわかりますが、「TP, FPなどの割合は？」についてはこのデータからは不明です。これらの値はどのようにして推定できるでしょうか？

ここで、モデルが正例と予測したうちの半数をわざとランダムに正負を判定して提出し、accurcy=0.5が得られたとします。このとき、モデル正負の反転によって以下のような予測値の（期待値の）変動が起こります。

* $TP^\prime \rightarrow TP / 2$
* $FP^\prime \rightarrow FP / 2$
* $TN^\prime \rightarrow TN + FP / 2$
* $FN^\prime \rightarrow FN + TP / 2$

これによって計算されるLBの予測値$a_2$は以下のようになります。

$$Na_2 = TP^\prime+TN^\prime = \frac{TP}{2} + TN + \frac{FP}{2} \tag{1}$$

一方、元のaccuracy=0.7のモデルのLBの予測値は、

$$Na_1 = TP + TN \tag{2}$$

です。(1), (2)はTP, TN, FPについての3元連立方程式で、変数3つに対して式の数が2つなので解けませんが、もう一つ独立な式を足してやれば解くことができます。具体的にはランダムに反転させる正例の割合を1/4などとします。

このように、

* モデルの予測値を意図的に編集して、一時独立な関係をいくつか作り、連立方程式を解く

というのが基本的なLB Probingの手法です。もちろん、このような一時独立な関係が作れるかどうかはコンペの評価メトリクスによるため、コンペごとに推定方法を考える必要があります。
また、この手法は編集前と編集後のLBスコアの差を使って推定しているので、そもそものLBスコアの精度が低いと、推定値の精度もそれによって低くなります。

従って、この手法が適用できるかどうかはコンペの設定に応じて適宜見極めが必要になります。

## 今までに使ったLB Probing手法の目的別まとめ

最後に、今まで私がKaggleコンペで使ったLB Probing手法についての一覧表を作りました。コンペごとの推定方法の詳細についてはリンク先のKaggleディスカッションで確認できます。

| 目的                                   | 推定した統計量                             | コンペ            | 手法の説明                                                                                                                                  |
| -------------------------------------- | ------------------------------------------ | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| より詳細な評価メトリクスを知る         | precision / recall                         | GBR[^1]           | https://www.kaggle.com/competitions/tensorflow-great-barrier-reef/discussion/302130                                                         |
|                                        | サブグループごとの部分スコア               | BirdCLEF 2022[^2] | https://www.kaggle.com/competitions/birdclef-2022/discussion/322419                                                                         |
|                                        | 任意の統計量（例: モデルの正例の予測比率） | BirdCLEF 2022     | https://www.kaggle.com/competitions/birdclef-2022/discussion/322606                                                                         |
| test dataにおけるラベルの分布を知る    | frameあたりのGT labelの割合                | GBR               | https://www.kaggle.com/competitions/tensorflow-great-barrier-reef/discussion/302156                                                         |
| 評価メトリクスのアルゴリズムを推定する | -                                          | BirdCLEF 2022     | 仮説から予測されるLBの推定値が実際のLBの観測値と一致することを確認する。https://www.kaggle.com/competitions/birdclef-2022/discussion/321883 |
| テストデータ中のリークの割合を推定する | テストデータ中の訓練データとの重複率       | 4sq[^3]           | https://www.kaggle.com/competitions/foursquare-location-matching/discussion/336047                                                          |

[^1]: https://www.kaggle.com/competitions/tensorflow-great-barrier-reef
[^2]: https://www.kaggle.com/competitions/birdclef-2022
[^3]: https://www.kaggle.com/competitions/foursquare-location-matching
