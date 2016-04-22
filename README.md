# vagrant-box.default

設定済みのvagrant boxです。  

## Usage

> 予め [Virtual Box](https://www.virtualbox.org/), [Vagrant](https://www.vagrantup.com/) をインストールしておくこと。

[リリース一覧](../../releases) から該当のリリースをえらんで、ファイルをダウンロードする。  

`package.box` を保存し、以下のコマンドを実行して box list に追加する。  
`Default` となっているところは任意の名前を指定できるが  
`Vagrantfile` で `Default` を指定している個所があるため、変更する場合は合わせて修正しておく必要がある。  

```bash
# boxへ追加
$ vagrant box add Default package.box
# 確認
$ vagrant box list
```

任意のディレクトリに `Vagrantfile` を配置する。  

コマンドラインやターミナルで `Vagrantfile` を配置したディレクトリに移動する。  

以下のコマンドを実行して、仮想マシンを起動する。

```bash
$ vagrant up
```

sshコマンドがインストールされている環境では `vagrant ssh` を  
そうでない場合は任意のsshクライアントアプリケーションを使用して仮想マシンにログインする。

|IP|port|
|:--|:--|
|localhost(127.0.0.1)|2222|

仮想マシンのユーザ情報は以下の通り。

|User|Pass|
|:--|:--|
|vagrant|vagrant|
|root|vagrant|

次回以降のログインはprivate keyを作成するなどでユーザ情報の入力を省けるようにしてもよい。

仮想マシンを終了する場合は以下のコマンドを実行する。

```bash
$ vagrant halt
```
