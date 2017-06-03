# demo_mnist
TensorFlowのデモ

mnistデータを使ったDeep Learningのデモ（2016年オープンキャンパス用）

# インストール

参考 Tensorflowのインストール手順 - Qiita http://qiita.com/box-man/private/6f9c3d0f2773b263d335

## AWS EC2に環境準備
* g2.2xlarge

GPUは使わないので、t2.microでもよい

## ubuntu 14.04

```
ssh -i プライベートキー ubuntu@tensor-test.niimilab.com.ap-northeast-1c

sudo locale-gen ja_JP.UTF-8
sudo ln -sf /usr/share/zoneinfo/Japan /etc/localtime

sudo apt-get update
sudo apt-get upgrade
sudo apt-get install git gcc make libreadline-dev zlib1g-dev libbz2-dev libsqlite3-dev libssl-dev
```

必要なファイルが分からないときは以下の作業で捜す

```
sudo apt-get install apt-file
sudo apt-file update
apt-file search zlib
apt-file search readline
apt-file search bzip2
apt-file search sqlite3
apt-file search openssl
```

## codaをインストールするために
参考 Ubuntu14.04にcuda 7.5をインストール - Qiita http://qiita.com/shinya_ohtani/items/f374ed0dd51737087369

```
wget http://jp.download.nvidia.com/XFree86/Linux-x86_64/367.35/NVIDIA-Linux-x86_64-367.35.run
sudo sh NVIDIA-Linux-x86_64-367.35.run -a --disable-nouveau

wget http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1404/x86_64/cuda-repo-ubuntu1404_7.5-18_amd64.deb
sudo dpkg -i  cuda-repo-ubuntu1404_7.5-18_amd64.deb
```

pythonでHello, TensorFlow!はうまくいったが、demo_mnistはエラーで落ちた。。。。

## Node.jsのインストール
### nvmのインストール

```
git clone https://github.com/creationix/nvm.git ~/.nvm && cd ~/.nvm && git checkout `git describe --abbrev=0 --tags`
```

~/.profileに下記を追加
```
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm
```
.bash_profileを適用
```
$ soruce ~/.bash_profile
```
### node.jsのインストール

バージョン確認
```
$ nvm ls-remote
```

### インストール（v6.3.1を使いました。）
```
$ nvm install v6.3.1
$ nvm use v6.3.1
```

pyenvをホームディレクトリ直下にインストール
```
$ git clone https://github.com/yyuu/pyenv.git ~/.pyenv
```

### pyenvのパスを通す
~/.profileに下記を追加
```
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"

eval "$(pyenv init -)"
```
```
pyenv install 2.7.12
pyenv global 2.7.12
```
```
# Ubuntu/Linux 64-bit, GPU enabled, Python 2.7 
# Requires CUDA toolkit 7.5 and CuDNN v4. For other versions, see "Install from sources" below.
$ export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/linux/gpu/tensorflow-0.9.0-cp27-none-linux_x86_64.whl
# Ubuntu/Linux 64-bit, CPU only, Python 2.7
$ export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/linux/cpu/tensorflow-0.9.0-cp27-none-linux_x86_64.whl

pip install --upgrade pip
pip install --upgrade $TF_BINARY_URL
```

一応、cudaがインストールされていれば、どちらでもインストールは成功する

画像用ライブラリ
```
pip install pillow
```

```
git clone https://github.com/b1013251/demo_mnist
cd demo_mnist
```
### モジュールのインストール
```
$ npm install
```

### サーバの起動
```
$ node index.js
```

### iptabesを使って、port80からport3000に転送
```
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 3000
```

### サーバ化

```
npm install foever
mkdir foever
vi development.json

{
    // Comments are supported
    "uid": "app",
    "append": true,
    "watch": true,
    "script": "index.js",
    "sourceDir": "/home/ubuntu/demo_mnist"
}
```

```
./node_modules/forever/bin/forever start demo_mnist/forever/development.json
```

起動はするし、トップページも見れるが、文字判断の途中で落ちているみたい
どうやら、index.jsで読んでいるpythonが終了する際にSIGKILLが飛ぶらしく、これをforeverが拾ってしまい、結果を返す前にnode.jsを再起動しているみたい

### sshセッション切れに対応
サーバ化がうまくいかないので、tmuxで運用する。

```
tmux
cd demo_mnist
node index.js
```

ターミナルを閉じても、node.jsが立ち上がりっぱなしになる
次に繋ぐときは、

```
tmux attach
```
