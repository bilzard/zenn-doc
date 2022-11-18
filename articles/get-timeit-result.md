---
title: "%timeitの測定結果を処理する"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["パフォーマンス計測", "python", "numpy", "pandas"]
published: true
published_at: 2022-11-19 09:00
---
# 概要

ipython notebookで`%timeit`の測定結果を分析する際、結果をpythonの変数から参照したくなる。ドキュメント[^1]によれば、`-o`オプションをつけると取得できる。

[^1]: https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit

# `TimeitResult` オブジェクトの構造

```python
result = %timeit -o [i ** 2 for i in range(10_000)]
```

`result`は`TimeitResult`クラスのオブジェクト[^2]。ドキュメントによると、以下のような属性を持つ[^3]。

* loops: 1回の計測における対象の処理の反復回数
* repeat: 計測回数
* best: 最も良かった測定結果
* worst: 最も悪かった測定結果
* all_runs: 各計測にかかった時間（単位: 秒）
* compile_time: 式のコンパイルにかかった時間（単位: 秒）
* average(readonly): 測定結果の平均値
* srdev(readonly): 測定結果の分散

注意すべきなのは、`all_runs`の結果は`loop`回の合計値であること。`all_runs`の結果から1回あたりの処理時間を計算するには、`loop`回数で割る必要がある。

[^2]: `IPython.core.magics.execution.TimeitResult`をインポートして`help()`にかければわかる。もっと簡単には、`result`オブジェクトを`help()`にかければわかる。
[^3]: 変数の意味はともかく何が格納されているかを知りたければ`vars(result)`で属性を列挙すればわかる。
# 計測例

実際に計測してみる。以下の5通りの書き方でfilter処理の速度を比較する。

1. `result[:] = filter(<filter_func>, xs)`
2. `result[:] = (x for x in xs if <filter_cond>)`
3. `result[:] = [x for x in xs if <filter_cond>]`
4. `result = xs[<filter_cond>]` (xs: numpy.array)
5. `result = xs[<filter_cond>]` (xs: pandas.DataFrame)

## 計測のコード


```python
%load_ext lab_black

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

plt.style.use("ggplot")
```


```python
sim_result = {}
N = 100_000

# filter
xs = np.random.randint(0, 256, N)
result = []
sim_result["filter"] = %timeit -o result[:] = filter(lambda x: x > 128, xs)

# tuple comprehension
xs = np.random.randint(0, 256, N)
result = []
sim_result["tuple comp."] = %timeit -o result[:] = (x for x in xs if x > 128)

# list comprehension
xs = np.random.randint(0, 256, N)
result = []
sim_result["list comp."] = %timeit -o result[:] = [x for x in xs if x > 128]

# filter with numpy
xs = np.random.randint(0, 256, N)
sim_result["numpy"] = %timeit -o result = xs[xs > 128]

# pandas
xs = pd.DataFrame(np.random.randint(0, 256, N))
sim_result["pandas"] = %timeit -o result = xs[xs > 128]
```

```python
data = []
for name, result in sim_result.items():
    data.append(
        {
            "name": name,
            "mean": result.average,
            "std": result.stdev,
            "loops": result.loops,
            "repeat": result.repeat,
        }
    )
data = pd.DataFrame(data)
data
```

|        name |     mean |          std | loops | repeat |
| ----------: | -------: | -----------: | ----: | -----: |
|      filter | 0.004671 | 1.955384e-05 |   100 |      7 |
| tuple comp. | 0.003846 | 1.717085e-05 |   100 |      7 |
|  list comp. | 0.003649 | 1.697157e-05 |   100 |      7 |
|       numpy | 0.000331 | 1.724407e-07 |  1000 |      7 |
|      pandas | 0.000600 | 4.628435e-07 |  1000 |      7 |


```python
_, ax = plt.subplots(figsize=(8, 3))
ax.bar(data["name"], data["mean"], yerr=data["std"], ecolor="black", capsize=10)
ax.set(xlabel="algorithm", ylabel="Execution Time (sec)")
plt.show()
```

![result](/images/simulate_filter_output.png)

という感じでいい感じにグラフにすることができた。
