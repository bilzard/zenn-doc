---
title: "各種filter演算の速度比較"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["パフォーマンス計測", "python", "numpy", "pandas"]
published: true
---
# listとnumpy arrayでfilter演算の速度を比較する

以下の5つの書き方で速度を比較した。

1. `result[:] = filter(<filter_func>, xs)`
2. `result[:] = (x for x in xs if <filter_cond>)`
3. `result[:] = [x for x in xs if <filter_cond>]`
4. `result = xs[<filter_cond>]` (xs: numpy array)
5. `result = xs[<filter_cond>]` (xs: pandas DataFrame)

## 速度比較のコード

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
sim_result["tuple comprehension"] = %timeit -o result[:] = (x for x in xs if x > 128)

# list comprehension
xs = np.random.randint(0, 256, N)
result = []
sim_result["list comprehension"] = %timeit -o result[:] = [x for x in xs if x > 128]

# filter with numpy
xs = np.random.randint(0, 256, N)
sim_result["numpy"] = %timeit -o result = xs[xs > 128]

# pandas
xs = pd.DataFrame(np.random.randint(0, 256, N))
sim_result["pandas"] = %timeit -o result = xs[xs > 128]
```

    4.9 ms ± 126 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
    3.93 ms ± 33.9 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
    3.7 ms ± 12.8 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)
    331 µs ± 207 ns per loop (mean ± std. dev. of 7 runs, 1,000 loops each)
    600 µs ± 536 ns per loop (mean ± std. dev. of 7 runs, 1,000 loops each)



```python
data = []
for sim, result in sim_result.items():
    data.append(
        {
            "name": sim,
            "mean": np.mean(np.array(result.all_runs) / result.loops),
            "std": np.std(np.array(result.all_runs) / result.loops),
        }
    )
data = pd.DataFrame(data)
data
```

|        name |     mean |          std |
| ----------: | -------: | -----------: |
|      filter | 0.004798 | 6.972129e-05 |
| tuple comp. | 0.003968 | 8.627254e-05 |
|  list comp. | 0.003746 | 2.608932e-05 |
|       numpy | 0.000332 | 3.192401e-07 |
|      pandas | 0.000631 | 8.205942e-07 |


```python
_, ax = plt.subplots(figsize=(8, 3))
ax.bar(data["name"], data["mean"], yerr=data["std"], ecolor="black", capsize=10)
ax.set(xlabel="algorithm", ylabel="Execution Time (sec)")
plt.show()
```

![output](/images/simulate_filter_output.png)

## 結果

全体的にはnumpyが圧倒的に早かった（リスト内包と比較しても10倍以上）。
list演算系ではリスト内包の書き方が最も早かった。
pandasは内部のデータ構造がnumpyであるが、さまざまなオーバーヘッド処理を行なっているためnumpyより遅くなる。今回のシミュレーションの条件ではnumpy配列よりも2.6倍遅かった。

## 参考

`%timeit -o`で計測結果を変数に取得できる。
https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit
