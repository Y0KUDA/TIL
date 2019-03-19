import from qiita  
https://qiita.com/Y0KUDA/items/3d3342ef08b28d5cda71

---
title: JuliaでCSV / DataFrameを扱う方法
tags: Julia DataFrame 機械学習 CSV データサイエンス
author: Y0KUDA
slide: false
---
# TL;DR
* 機械学習や統計で基本となるCSVデータの扱い方を書きます。
* Juliaでは、CSVをDataFrameとして扱うことができます。
  * RやPythonでCSVを扱ったことがあればすぐに使いこなすことができます。

# 下準備編
### 専用パッケージの追加
このコードを実行することでCSVを扱うために必要なパッケージが追加されます。
スクリプトでも実行可能ですが、Repl環境で実行することをおすすめします。

```julia
using Pkg
Pkg.add("CSV")
```

### パッケージのロード
このコードを実行することでCSVパッケージがロードされます。
初回はビルドが走るので若干時間がかかります。

```julia
using CSV
```

# 本編
### 前提条件
解説のためにこのCSVデータを扱います。タイタニック乗客のデータです。
http://web.stanford.edu/class/archive/cs/cs109/cs109.1166/stuff/titanic.csv
![csv.png](https://qiita-image-store.s3.amazonaws.com/0/275289/57520f3a-8dff-2000-670b-39d27462e9fa.png)
詳細を知りたい方は以下リンクを参照してください。
http://web.stanford.edu/class/archive/cs/cs109/cs109.1166/problem12.html

### CSVをデータフレームとしてロードする
`CSV.read(ファイルパス)`を使います。
次のコードを実行することで、`titanic`にCSVがデータフレームとして代入されます。

```julia
using CSV
titanic = CSV.read("./titanic.csv") # ロード
```

### データフレームをCSVとしてセーブする
`CSV.write(ファイルパス,dataframe)`を使います。
次のコードを実行することで`output.csv`という名前でセーブできます。

```julia
using CSV
titanic = CSV.read("./titanic.csv")
CSV.write("./output.csv",titanic) # セーブ
```



### 列を指定して配列として取得する
#### 列番号で指定
`dataframe[列番号]`で取得できます。
※ Juliaはインデックス番号が1から始まるので注意。
次のコードを実行することで1列目の`Survived`を配列として取得することができる。

```julia
using CSV
titanic = CSV.read("./titanic.csv")
survived = titanic[1]
```

#### 列名で指定
`dataframe.列名`で取得できます。
次のコードを実行することで`Sex`の列を配列として取得することができる。

```julia
using CSV
titanic = CSV.read("./titanic.csv")
sex = titanic.Sex
```

`dataframe[:列名]`のように書くこともできます。

```julia
using CSV
titanic = CSV.read("./titanic.csv")
sex = titanic[:Sex]
```

`dataframe[Symbol("列名")]`のように書くこともできます。
列名に空白が入っている場合に便利です。

```julia
using CSV
titanic = CSV.read("./titanic.csv")
parentChildren_Aboard = titanic[Symbol("Parents/Children Aboard")]
```

### 列を指定して切り抜く・並び替える
`dataframe[[:列名1,:列名2,:列名3,...]]`で列の切り抜き・並び替えができます。
次のコードで`Sex,Name,Age`のデータフレームを作成できます。

```julia
using CSV
titanic = CSV.read("./titanic.csv")
titanic2 = titanic[[:Sex,:Name,:Age]]
```

### 行を指定して切り抜く・並び替える
`dataframe[[行番号1,行番号2,行番号3,...],:]`で行の切り抜き・並び替えができます。
次のコードで3行目、4行目、8行目を切り抜いたデータフレームを作成できます。

```julia
using CSV
titanic = CSV.read("./titanic.csv")
titanic2 = titanic[[3,4,8],:]
```

ある行からある行までまとめて切り抜く場合は`dataframe[行番号1:行番号2,:]`のように書くこともできます。
次のコードで10行目から20行目までを切り抜いたデータフレームを作成できます。

```julia
using CSV
titanic = CSV.read("./titanic.csv")
titanic2 = titanic[10:20,:]
```

### 条件に合う行を切り抜く
`dataframe[行数分のtrue/false配列,:]`でtrueの行だけ切り抜きます。
次のコードで`Sex`が`female`の行だけ切り抜いたデータフレームを作成できます。
※Juliaでは、`f.(array)`のように`.`を付けて関数を実行すると`map(f,array)`のように動作します。

```julia
using CSV
titanic = CSV.read("./titanic.csv")

function isFemale(sex)
  return sex == "female"
end

female = titanic[isFemale.(titanic.Sex),:]
```
このケースではもっと簡単に、次のように書くこともできます。

```julia
using CSV
titanic = CSV.read("./titanic.csv")
female = titanic[titanic.Sex .== "female",:]
```

`Age`が20以上の行を切り抜く場合は次のように書きます。

```julia
using CSV
titanic = CSV.read("./titanic.csv")
over20 = titanic[titanic.Age .>= 20,:]
```


### データの書き換え
`dataframe[行番号,:列名] = 書き換えたい値`で値を書き換えることができます。
次のコードでは一行目の`Sex`を`boy`に書き換えます。

```julia
using CSV
titanic = CSV.read("./titanic.csv")
titanic[1,:Sex] = "boy"
```
例えば、`Sex`の`female`を`girl`、`male`を`boy`に書き換えるには次のように書きます。

```julia
using CSV
titanic = CSV.read("./titanic.csv")
titanic[titanic.Sex .== "male",:Sex] = "boy"
titanic[titanic.Sex .== "female",:Sex] = "girl"
```

### ソート
`CSV.sort(dataframe,:列名)`でデータフレームをソートすることができます。
次のコードでは`Age`を基準にソートします。

```julia
using CSV
titanic = CSV.read("./titanic.csv")
ageSorted = CSV.sort(titanic,:Age)
```






### 行数、列数を取得する
`size(dataframe)`で取得できます。
列数は`size(dataframe)[1]`、行数は`size(dataframe)[2]`です。

```julia
using CSV
titanic = CSV.read("./titanic.csv")
row = size(titanic)[1]
col = size(titanic)[2]
```


### 簡単な情報を表示する
`CSV.describe(dataframe)`で各列の中央値、平均値などの簡単な情報を確認できます。

```julia
using CSV
titanic = CSV.read("./titanic.csv")
CSV.describe(titanic)
```
出力
![スクリーンショット (12).png](https://qiita-image-store.s3.amazonaws.com/0/275289/233357d4-272f-aed9-abdc-111211d91764.png)

### クロス集計
`CSV.by(dataframe,:グルーピングする列名,:列名1 => 関数1,:列名2 => 関数2...)`で行えます。
`関数1`にはグループの`列名1`から切り出した値の配列を引数として与えられます。
同様に`関数2`にはグループの`列名2`から切り出した値の配列を引数として与えられます。
例えば、性別による生存者、死者の集計は次のコードで行えます。

```julia
using CSV
titanic = CSV.read("./titanic.csv")
by( titanic,
    :Sex,                         # Sexでグルーピングする
    :Survived => sum,             # Survivedが1の場合は生存なので、合計すると生存者数になる
    :Survived => x -> sum(x.==0)  # Survivedが0の場合は死者なので、0の数を合計すると死者数になる
)
```
出力
![スクリーンショット (14).png](https://qiita-image-store.s3.amazonaws.com/0/275289/e2fd483f-f9ec-2f9d-60d2-b9f4693ddca4.png)

### 結合
`vcat(dataframe1,dataframe2)`で縦方向に結合できる。
`hcat(dataframe1,dataframe2)`で横方向に結合できる。



## 最後に
「dplyrないの？」「LINQ使いたい」って方はDataFramesMetaを使うと気持ちよくなれるかもしれません。
https://github.com/JuliaData/DataFramesMeta.jl
これについても書きたかったですが、力尽きました。

---

「●●が入ってないやん」
「●●が間違ってる」
「●●の方がスマート」
等のマサカリ大歓迎です。

「良かった」
「悪かった」
「ﾀﾋね」
等の感想いただけると大変うれしいです。











