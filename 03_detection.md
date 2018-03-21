# Detection model(SSD) prediction
* 参考: 「SSD: Single Shot MultiBox Detector 高速リアルタイム物体検出デモをKerasで試す」(https://qiita.com/PonDad/items/6f9e6d9397951cadc6be)

## libgccのコピー
cv2をインポートする際に
```shell
ImportError: /home/ubuntu/anaconda3/envs/tensorflow_p36/bin/../lib/libstdc++.so.6: version `CXXABI_1.3.8' not found (required by /home/ubuntu/anaconda3/envs/tensorflow_p36/lib/python3.6/site-packages/../../././libicui18n.so.58)
```
と出るのでlibstdc++をコピーしておく。
```shell
$ cp /usr/lib/x86_64-linux-gnu/libstdc++.so.6 /home/ubuntu/anaconda3/envs/tensorflow_p36/lib
```

```shell
$ source activate tensorflow_p36
$ python
>>> import cv2
```
でエラーがでないことを確認する。

## SSDのKeras実装をダウンロード
```shell
$ cd
$ git clone https://github.com/rykov8/ssd_keras.git
```

## Keras2用にスクリプトを修正

* 公開されているSSDのKeras実装はKeras2非対応なので
https://gist.github.com/anonymous/4c3105119a233cb33926651c3ea1966c
から修正版のssd.pyをダウンロードし、差し替える。

* ssd_layers.pyの
```python
    def get_output_shape_for(self, input_shape):
```
を
```python
    def compute_output_shape(self, input_shape):
```
に変更する。

https://github.com/cory8249/ssd_keras.git

をcloneするのが正解？

## 学習済みモデルのダウンロード
学習済みモデルが
https://mega.nz/#F!7RowVLCL!q3cEVRK9jyOSB9el3SssIA
に公開されているので「weights_SSD300.hdf5
」をダウンロードし、ssd_kerasディレクトリに保存する。
```shell
$ cd
$ cd ssd_keras
$ unzip SSD.zip
$ mv SSD/weights_SSD300.hdf5 .
```

## Jupyter notebookの起動

* すでにJupyter notebookが起動済みの場合は終了する
```shell
$ ps -ef | grep jupyter
$ kill (プロセスID)
```

* ssd_kerasをホームディレクトリにしてJupyter notebookを起動する
```shell
$ cd
$ cd ssd_keras
$ jupyter notebook
```

## Notebookの作成
* Jupyter notebookの「New」から「Environment(conda_tensorflow_p36)」を選択する
* 適当な名前(SSD_predictionなど)をつける
* 画像をサーバ側に移しておく
* 「Keras Applications」(https://keras.io/ja/applications/)の「Classify ImageNet classes with ResNet50」を実行する

