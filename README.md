# vagrant-box.default

設定済みのvagrant boxです。  

## Usage

> 予め Virtual Box, Vagrant をインストールしておくこと。

[リリース一覧](,/releases) から該当のリリースをえらんで、ファイルをダウンロードする。  

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
