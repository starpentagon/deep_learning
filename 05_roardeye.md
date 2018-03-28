# RoadEye用モデル

## データセット(20180323版)
1. 20180322提供分(NTTDアノテーション実施分)
2. 03/18, 03/23 NTTD関西アノテーション分

## 分類ラベル名(20180323版)
スマソ委託時の分類ラベルを付与
1. 大型自動車
2. 普通自動車
3. 自動二輪
4. 自転車
5. 歩行者
6. その他

* 「自動二輪」の125cc未満/以上の判定は対象外
* 「複数物体」は対象外
* NTTD関西実施分は分類ラベル名が統一されていないので修正（./roadeye/01_dir_rename.ipynb）

# 分類ラベルの基礎統計
## 動体数
|分類ラベル|動体数|
|:-------|----:|
|大型自動車|502|
|普通自動車|629|
|自動二輪  | 13|
|自転車   | 25|
|歩行者   |  1|
|その他   |  3|

* 歩行者、その他は動体数が少ないため対象外とする

## 画像数
|分類ラベル|動体数|
|:-------|----:|
|大型自動車|3212|
|普通自動車|4245|
|自動二輪  | 132|
|自転車   | 439|
|歩行者   |   5|
|その他   |  13|

# 学習／評価データに分割

* 動体単位で学習／評価データに分割（分割比率 学習:評価=8:2）
  * 学習データ: 動体数: 935, 画像数: 6,372
  * 評価データ: 動体数: 234, 画像数: 1,643

# 画像分類
* Xceptionの最終層をglobal average poolingしたベクトルを特徴量として抽出
* 2048次元
* 学習用画像数: 6,372
* 評価用画像数: 1,643

# SVMによる画像分類(OneVSRestClassifier)

## ハイパーパラメタサーチ
* 3-fold cross validationでハイパーパラメタサーチ

### RBF kernel
* C = (0.0001, 1, 10000), gamma = (0.0001, 0.01)
  * 12.5min
  * Best: C = 10000, gamma = 0.01, avg-precision = 94.7%
* C = (100, 10000, 1000000), gamma = (0.001, 0.01, 0.1)
  * 26.1min
  * Best: C = 100, gamma = 0.01, avg-precision = 94.7%

* best paramでrefit
  * 学習データ precision: 99.8%

  |fact/pred| big_car | car | motorcycle  | bicycle |
  |:--|--------:|----:|------------:|--------:|
  |big_car|2583|5|0|0|
  |car|8|3331|0|0|
  |motorcycle|0|0|83|0|
  |bicycle|0|0|0|362|

  * 評価データ precision: 94.0%

  |fact/pred| big_car | car | motorcycle  | bicycle |
  |:--|--------:|----:|------------:|--------:|
  |big_car|559|52|0|0|
  |car|26|879|0|1|
  |motorcycle|1|12|29|7|
  |bicycle|0|0|0|77|

### Linear kernel
* C = np.logspace(-4, 4, 9)
  * 8.1min
  * Best: C = 0.1, avg-precision = 94.6%
* best paramでrefit
  * 学習データ precision: 98.4%

  |fact/pred| big_car | car | motorcycle  | bicycle |
  |:--|--------:|----:|------------:|--------:|
  |big_car|2538|50|0|0|
  |car|49|3290|0|0|
  |motorcycle|0|0|83|0|
  |bicycle|0|0|0|362|

  * 評価データ precision: 94.0%

  |fact/pred| big_car | car | motorcycle  | bicycle |
  |:--|--------:|----:|------------:|--------:|
  |big_car|560|51|0|0|
  |car|31|875|0|0|
  |motorcycle|0|11|35|3|
  |bicycle|0|2|1|74|

# 物体分類
* 動体ごとに画像ラベルの多数決で動体ラベルを決定
* precision: 94.9%

|fact/pred| big_car | car | motorcycle  | bicycle |
|:--|--------:|----:|------------:|--------:|
|big_car|89|8|0|0|
|car|3|123|0|0|
|motorcycle|0|1|4|0|
|bicycle|0|0|0|6|
