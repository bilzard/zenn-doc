---
title: "Paper: An Alternative Cross Entropy Loss for Learning-to-Rank"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [learningtorank, paper]
published: false
---

## 概要

本論文[^1]では、リストワイズのランキング学習手法であるListNetの損失関数を改良した$\mathrm{XE}_\mathrm{NDCG}$を提案する。

[^1]: https://arxiv.org/abs/1911.09798

## 提案手法の特徴

* LambdaRankは評価指標との相関性を経験的に示したに過ぎないが、本論文では損失関数と評価指標の一貫性について理論的な根拠を示している。
* Lambdarankが全てのドキュメントのペアについて損失を計算するため$O(n^2)$の計算コストを持つが、提案手法（およびListNet）は個々のドキュメントのスコアについて損失を計算するので$O(n)$の計算コストをもつ。
* 実験的に、ListNetを統計的に有意に上回り、LambdaRankと同等の性能を持つことが示された。また、提案手法はLambdaRankと比べて人工的に付加されたノイズにも強いことが示された。

## ListNetとXE_NDCGの損失関数の違い

ListNetがSoftmaxで確率に正規化したドキュメントの評価値の分布どうしのクロスエントロピーを最大化するのに対し、提案手法は正解ラベルの分布がListNetと異なり、NDCGに似た重みづけになっている。$\bm{\gamma}$はパイパーパラメータで、ドキュメントにごとに固有の値を持つ。

ListNet:
$$\rho(f_i) = \frac{e^{f_i}}{\sum_{j=1}^m e^{f_j}}, \phi(y_i) = \frac{e^{y_i}}{\sum_ {j=1}^m e^{y_j}}$$

XE_NDCG:
$$\rho(f_i) = \frac{e^{f_i}}{\sum_{j=1}^m e^{f_j}}, \phi(y_i; \bm{\gamma}) = \frac{2^{y_i} - \gamma_i}{\sum_ {j=1}^m 2^{y_j} - \gamma_j}$$

## その他周辺的な情報

* LightGBMが`rank_xendcg`として公式にサポートしている[^2]。
* 著者による解説動画[^3]が公開されている。
* LightGBMにおける実装のコード[^4]

[^2]: https://lightgbm.readthedocs.io/en/latest/Parameters.html
[^3]: https://www.youtube.com/watch?v=BjbSuEEBau8
[^4]: https://github.com/microsoft/LightGBM/blob/master/src/objective/rank_objective.hpp#L285-L363