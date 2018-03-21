# 環境構築

## 環境構築 on AWS
* リージョン: 東京
* AMI: Deep Learning AMI (Ubuntu) Version 5.0
* インスタンス: p2.xlarge
* ストレージ: 100GB
* セキュリティグループ: dnn_deploy_sg
    * TCP(SSH): 22
    * TCP(Jupyter): 8888
    * TCP(TensorBoard): 6006
* キーペア: model_deploy.pem
    * Puttyからアクセスできるよう秘密鍵ファイル(ppk)を作成しておく

## インスタンスの環境構築
### Puttyからアクセス
* ホスト名: ubuntu@(パブリックDNS名)
* SSH -> 認証 -> 認証のための秘密鍵: 秘密鍵ファイルのパスを指定する

インスタンスにアクセスできることを確認する。

### GPUの確認
```shell
$ nvidia-smi
```
でGPU(p2の場合はTesla K80, p3の場合はVolta V100)が認識されていることを確認する

### Ubuntu環境のupdate
```shell
$ sudo apt update
$ sudo apt upgrade
```

### Jupyter notebookの設定
1. パスワード作成
```shell
$ ipython3
```
```python
from notebook.auth import passwd
passwd()
quit()
```
パスワードは「社名を小文字」。表示されるsha1をメモしておく。
('sha1:dc2d59cc7ca3:74ce705df090316be32834b76c7dea7a4ad78912')

2. Jupyter設定ファイルの編集

```shell
$ vi .jupyter/jupyter_notebook_config.py
```

c.NotebookApp.ipを変更
```python
c.NotebookApp.ip = '0.0.0.0'
```

c.NotebookApp.passwordを変更
```python
c.NotebookApp.password = u'sha1:dc2d59cc7ca3:74ce705df090316be32834b76c7dea7a4ad78912'
```

c.NotebookApp.portを変更
```python
c.NotebookApp.port = 8888
```

c.NotebookApp.open_browserを変更
```python
c.NotebookApp.open_browser = False
```

3. Jupyterを起動
```shell
$ jupyter notebook
```

4. クライアントからアクセス

クライアントのブラウザから
```text
http://(パブリックDNS名):8888
```
にアクセスし、パスワード入力画面が表示されることを確認する。

「Jupyter notebookの設定」で作成したパスワードを入力してアクセスできることを確認する。

アクセスできない場合はポート番号が8888以外になっていないか（Jupyter notebookを複数起動するとポート番号がインクリメントされていく）確認する。
```shell
$ ps -ef | grep jupyter
```
