---
title: "Pocketfft: C++のFFTライブラリの使用方法"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cpp", "fft"]
published: true
---

## Pocketfft とは

https://github.com/mreineck/pocketfft

Pocketfft は、C++ 用の FFT（高速フーリエ変換）ライブラリである。これは [FFTPACK](https://casa.nrao.edu/aips2_docs/scimath/implement/Mathematics/FFTPack.html) を改良したもので、たとえば Python の [Numpy で採用されている](https://numpy.org/devdocs/release/1.17.0-notes.html)。

Pocketfft の特徴は次の通りである。

- **ヘッダー**のみ（`pocketfft_hdronly.h`）
- 3-clause BSD ライセンス
- `float`、`double`、`long double` 型をサポート
- 2 の冪乗でないフレームサイズもサポート
- **多次元変換**および離散コサイン変換と離散サイン変換をサポート
- 完全複素数および半複素（複素数から実数または実数から複素数）FFT をサポート

Pocketfft はヘッダーのみのライブラリなので、導入が容易である。有名な [FFTW](https://www.fftw.org/) は GPL-2.0 ライセンスなので、Pocketfft はライセンスの観点で比較的採用しやすい。

また、パフォーマンスの点において、他 FFT ライブラリと比較して、Pocketfft の優位性が次の検証で示されている。ただし、全てのユースケースで Pocketfft がベストとは限らないので、自身のプロジェクトと照らし合わせる必要がある。

https://github.com/project-gemmi/benchmarking-fft

この記事では、Pocketfft の使い方を [pocketfft_demo.cc](https://github.com/mreineck/pocketfft/blob/cpp/pocketfft_demo.cc) を元に解説する。

## インストール

Pocketfft はヘッダーのみのライブラリであるため、`pocketfft_hdronly.h` をプロジェクト内の適切なディレクトリに配置し、インクルードするだけで使用できる。

```cpp
#include "pocketfft_hdronly.h"
```

名前空間は `pocketfft` である。

```cpp
using namespace pocketfft;
```

## 主要な関数

- `c2c`: 複素数から複素数への FFT
- `r2c`: 実数から複素数への FFT
- `c2r`: 複素数から実数への FFT
- `r2r_fftpack`: 実数から半複素数への FFT（FFTPACK スタイル）
- `r2r_separable_hartley`: 分離可能なハートレイ変換
- `dct`/`dst`: 離散コサイン変換/離散サイン変換

## コンフィグ

- `POCKETFFT_CACHE_SIZE`: FFT プランの LRU キャッシュサイズを制御する
- `POCKETFFT_NO_VECTORS`: CPU ベクトル命令を無効にする
- `POCKETFFT_NO_MULTITHREADING`: マルチスレッディングを無効にする

## 使用例

### 基本的な１次元 FFT

以下は、実数配列から複素数配列への FFT の例である。複素数同士で実行する場合は `c2c` を使えばよい。また、`double` 以外を使用する場合は、単に `double` や `1.` などの箇所を変更する。

```cpp:demo1.cpp
#include <complex>
#include <cmath>
#include <iostream>
#include <random>
#include "pocketfft_hdronly.h"

using namespace pocketfft;

constexpr size_t N = 2048; // frame size

double mse(double X[], double Y[])
{
    double sum = 0.0;
    for (size_t i = 0; i < N; ++i)
        sum += pow(X[i] - Y[i], 2);
    return sum / (double)N;
}

int main()
{
    // 1-dim
    shape_t shape{N};
    shape_t axes = {0};
    stride_t stride_in = {sizeof(double)};
    stride_t stride_out = {sizeof(std::complex<double>)};

    // demo data
    double data[N];
    std::random_device rd;
    std::mt19937 gen(rd());
    std::uniform_real_distribution<double> distribution(0.0, 100.0);
    for (auto &d : data)
        d = distribution(gen);
    // FFT data
    std::vector<std::complex<double>> res(N);
    double inv_data[N];

    // FFT
    r2c(shape, stride_in, stride_out, axes, FORWARD, data, res.data(), 1.);
    // iFFT
    c2r(shape, stride_out, stride_in, axes, BACKWARD, res.data(), inv_data, 1. / (double)N);

    // mse error
    std::cout << mse(data, inv_data) << std::endl;

    return 0;
}
```

この例では、`shape_t` および `stride_t` を使用してデータの寸法とメモリレイアウトを定義する。`r2c`、`c2r`、`c2c` で FFT を実行する。方向は `FORWARD`/`BACKWARD` にて指定する。

逆方向に FFT をするとき、元のデータと一致させるためにデータ数でそれぞれ割る必要がある。これは `c2r` の `1./(double)N` に相当する。

g++ のコンパイル例は次の通りである。`-pthread` をつける必要がある。

```
$ g++ demo1.cpp -pthread
```

### ２次元以上の FFT

2 次元 FFT の例を示す。

```cpp:demo2.cpp
#include <complex>
#include <cmath>
#include <iostream>
#include <random>
#include "pocketfft_hdronly.h"

using namespace pocketfft;

constexpr size_t N = 2048; // frame size of 1-dim

double mse(std::vector<double> &X, std::vector<double> &Y)
{
    double sum = 0.0;
    for (size_t i = 0; i < N * N; ++i)
        sum += pow(X[i] - Y[i], 2);
    return sum / (double)(N * N);
}

int main()
{
    // 2-dim
    shape_t shape{N, N};
    shape_t axes;
    for (size_t i = 0; i < shape.size(); i++)
        axes.push_back(i);
    stride_t stride_in(shape.size()), stride_out(shape.size());
    size_t tmp_in = sizeof(double);
    size_t tmp_out = sizeof(std::complex<double>);
    for (int i = shape.size() - 1; i >= 0; i--)
    {
        stride_in[i] = tmp_in;
        tmp_in *= shape[i];
        stride_out[i] = tmp_out;
        tmp_out *= shape[i];
    }

    // demo data
    std::vector<double> data(N * N);
    std::random_device rd;
    std::mt19937 gen(rd());
    std::uniform_real_distribution<double> distribution(0.0, 100.0);
    for (auto &d : data)
        d = distribution(gen);
    // FFT data
    std::vector<std::complex<double>> res(N * N);
    std::vector<double> inv_data(N * N);

    // FFT
    r2c(shape, stride_in, stride_out, axes, FORWARD, data.data(), res.data(), 1.);
    // iFFT
    c2r(shape, stride_out, stride_in, axes, BACKWARD, res.data(), inv_data.data(), 1. / (double)(N * N));

    // mse error
    std::cout << mse(data, inv_data) << std::endl;

    return 0;
}
```

1 次元からの主な変更点は次の通りである。

- `shape{N}` → `shape{N, N}`
- `axes = {0}` → `axes = {0, 1}`
- `stride = {tmp}` → `stride = {tmp*N, tmp}`

また、FFT で使うデータは 1 次元であることに注意が必要である。たとえば、元が 2 次元のデータであれば、次のように変換する。

```cpp
for (size_t y = 0; y < N; y++)
{
    for (size_t x = 0; x < N; x++)
        tmp[x + y * N] = data[x][y];
}
```

## マルチスレッド

FFT の実行時には、パラメータ `nthreads` で使用するスレッドの数を指定できる。デフォルトは `nthreads=1` である。次の例は `4` にしたものである。

```cpp
r2c(shape, stride_in, stride_out, axes, FORWARD, data, res.data(), 1., 4);
c2r(shape, stride_out, stride_in, axes, BACKWARD, res.data(), inv_data, 1. / N, 4);
```

## 参考文献

- [GitHub, mreineck/pocketfft](https://github.com/mreineck/pocketfft)
- [GitHub, project-gemmi/benchmarking-fft](https://github.com/project-gemmi/benchmarking-fft)
- [TadaoYamaoka の開発日記、pocketfft を使ってみる](https://tadaoyamaoka.hatenablog.com/entry/2023/01/30/001600)
