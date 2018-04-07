# Segmentation model(FCN) prediction

## インストール
* python2が前提になっていることに注意
### keras-contrib for DenseNet based model
```shell
$ source activate tensorflow_p27
$ git clone https://github.com/ahundt/keras-contrib.git -b densenet-atrous
$ cd keras-contrib
$ sudo python setup.py install
```

### MS COCO API
* Keras-FCNのREADME.mdのパスは誤っている
* 「Microsoft COCO が提供するデータとAPIを使用すると画像解析が簡単に出来た」(https://qiita.com/GushiSnow/items/5a4948e07e3f4a77d895)を参考
```shell
$ cd ..
$ git clone https://github.com/cocodataset/cocoapi.git
$ cd cocoapi/PythonAPI
$ conda install cython
$ conda install scikit-image
```
* gccでのbuildが失敗(pthread, cがないと言われる)
```shell
$ cd /home/ubuntu/anaconda3/envs/tensorflow_p27/lib
$ ln -s /lib/x86_64-linux-gnu/libpthread.so.0 libpthread.so
$ ln -s /lib/x86_64-linux-gnu/libc.so.6 libc.so
$ make
$ make install
$ python setup.py install
```

### MS COCOダウンロード
* cocoapiのホームディレクトリに移動して画像とアノテーションをダウンロードする
```shell
$ wget http://images.cocodataset.org/zips/val2017.zip
$ unzip val2017.zip
$ mv val2017 images
$ wget http://images.cocodataset.org/annotations/annotations_trainval2017.zip
$ unzip annotations_trainval2017.zip
```

### MS COCOの確認
* cocoapi/PythonAPIをホームディレクトリにしてjupyter notebookを起動する
* 「pycocoDemo.ipynb」を開く
* Kernel -> Change Kernel -> Environment(conda_tensorflow_p36)を選択し実行できることを確認する

## FCNのKeras実装をダウンロード
```shell
$ git clone https://github.com/aurora95/Keras-FCN.git
$ git clone https://github.com/ahundt/tf-image-segmentation.git -b Keras-FCN
```

## Datasetのダウンロード
```shell
$ git clone https://github.com/ahundt/tf-image-segmentation.git -b Keras-FCN
```

## Automated Pascal VOC Setup
```shell
$ pip install sacred
$ export PYTHONPATH=$PYTHONPATH:~/deep_learning/keras_fcn/tf-image-segmentation
$ cd ./tf-image-segmentation/tf_image_segmentation/recipes/pascal_voc/
python data_pascal_voc.py pascal_voc_setup
```

## Training and testing
```shell
$ cd /home/ubuntu/deep_learning/keras_fcn/Keras-FCN
$ cd utils
$ python transfer_FCN.py ResNet50
$ cd ..
$ export PYTHONPATH=$PYTHONPATH:~/deep_learning/keras_fcn/keras-contrib
$ python train.py
```
* Keras 2.1.4だとimportエラー。Keras 2.0.8だとimportエラーは解消されるが「SegDirectoryIterator object is not an iterator」というエラーが出る
  * いったん中断
  * keras-contribをpipで最新版を入れたら解消
  * が、GPUを使ってくれない

## Evaluating
```shell
$ python evaluate.py
```
とすると「version `GOMP_4.0' not found」とエラー
```shell
$ cd /home/ubuntu/anaconda3/envs/tensorflow_p36/lib
$ mv libgomp.so libgomp.so.old
$ ln -s /usr/lib/gcc/x86_64-linux-gnu/5/libgomp.so .
$ ln -s /usr/lib/gcc/x86_64-linux-gnu/5/libgomp.so libgomp.so.1
```
とすると次は「`GLIBCXX_3.4.22' not found」とエラー
```shell
$ mv libstdc++.so.6 libstdc++.so.6.old
$ ln -s /usr/lib/gcc/x86_64-linux-gnu/5/libstdc++.so libstdc++.so.6
```
これでもエラー。どうやらデフォルトで入っているlibstdc++が古いらしい。
```shell
sudo add-apt-repository ppa:ubuntu-toolchain-r/test 
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
```
とするとエラー解消。

(pipでkeras-contribを入れるので不要)Pythonのパスを追加しておく。Keras-FCN/models.pyの先頭に

```python
import sys
sys.path.append('/home/ubuntu/deep_learning/keras_fcn/keras-contrib/keras_contrib')
```

を追加

```shell
ImportError: cannot import name '_postprocess_conv3d_output'
```
というエラーがでる。Kerasのバージョンが低い(2.1.4)のが原因っぽい。
```shell
$ conda install keras
```
で最新版をインストール。v2.1.5にしたがエラー解消せず。
結局
```shell
$ pip install git+https://www.github.com/keras-team/keras-contrib.git
```
でインストール

evaluate.pyにバグがあるので104行目の前に
```python
  data_suffix = '.jpg'
```
を追加

* 事前学習済みのモデルがみつけられず＆学習がGPU使えず遅い

# FCN_via_keras

Chainer前提なので
```shell
$ source activate chainer_p36
```
PILが入っていないと怒られるので
```shell
$ conda install pillow
```
cv2が入っていないと怒られるので
```shell
$ conda install -c https://conda.binstar.org/jjhelmus opencv
```
kerasが入っていないと怒られるので
```shell
$ conda install keras
```
がkerasのバージョンがあっておらずエラー

# Mask R-CNN

* GitHubレポジトリ(https://github.com/matterport/Mask_RCNN)のInstallationに従う
* Python3, TensorFlow, Keras前提なので
```shell
$ source activate tensorflow_p36
```
としておく

* レポジトリのclone
```shell
$ git clone https://github.com/matterport/Mask_RCNN.git
```
* 学習済みモデルのダウンロード
```shell
$ cd Mask_RCNN
$ wget https://github.com/matterport/Mask_RCNN/releases/download/v2.0/mask_rcnn_coco.h5
```
* Jupyter notebookを立ち上げる
```shell
$ jupyter notebook
```
* demo.ipynbを実行する
  * 画像によってはkernelが死ぬことがある。まだ、動作は不安定
