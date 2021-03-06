# vagrant-box.default

[CentOS6.5(64bit)](https://github.com/2creatives/vagrant-centos/releases/download/v6.5.3/centos65-x86_64-20140116.box) をベースとした、設定済みのvagrant boxです。  

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

## Setup

導入済みアプリケーションは以下の通り（導入に必要なパッケージもインストール済み）。

- Apache(httpd)
- MySQL
- php(5.3.3)
- rbenv
- ruby(2.0.0, 2.1.7, 2.2.4, global=2.2.4)
- gem rails
- gem bundler

ポートフォワーディングで以下を設定しているので  
アプリケーションが起動していればホストOS側からのアクセスが可能。

|HostOS|GuestOS|Usage|
|:--|:--|:--|
|2222|22|ssh|
|8080|80|httpd|
|3030|3000|webrick(rails)|

## Warning

### rails

#### webサーバ

railsアプリケーションをwebrick以外のwebサーバで動作させたい場合は  
追加で以下のようなアプリケーションを選択してインストールする。  

- phusion passenger(with httpd)
- nginx and unicorn
- puma

#### bundler

`bundle install`する場合は共有ディレクトリ（/vagrant）にインストールしてはならない。  
（インストールに失敗するgemが出てくる）  

従って以下のコマンドのオプション指定は **間違い** となる。  

```bash
$ bundle install --path=vendor/bundle
```

正しくは以下のようにオプションを指定して実行する。  
（パスの指定は共有ディレクトリでなければどこでもよい）

```bash
# グローバルにインストールする場合
$ bundle install
# インストールディレクトリを指定する場合
$ bundle install --path=~/vendor/bundle
```

## Tips

### rails

#### environment

起動する環境モードが固定されている場合は  
`~/.bash_profile`などで環境変数`RAILS_ENV`を追加しておくとよい。  

```bash
$ echo export RAILS_ENV=production >> ~/.bash_profile
$ source ~/.bash_profile
# 確認
$ echo $RAILS_ENV
# => production
```

#### WEBrickを使ったSSL

証明書を発行する。  

```bash
$ mkdir ~/.ssl
$ openssl req -new -newkey rsa:2048 -sha1 -days 365 -nodes -x509 -keyout ~/.ssl/localhost.key -out ~/.ssl/localhost.crt
```

`config/ssl.rb`を作成し、以下の内容を記述する。  
ポート番号の重複を回避するため、`Port`は33000を設定する。  
（`Vagrantfile`の`forwarded_port`も設定しておく）

```ruby
require 'rubygems'
require 'rails/commands/server'
require 'rack'
require 'webrick'
require 'webrick/https'

module Rails
  class Server < ::Rack::Server
    def default_options 
      super.merge({
        Port: 33000,
        SSLEnable: true,
        SSLVerifyClient: OpenSSL::SSL::VERIFY_NONE,
        SSLPrivateKey: OpenSSL::PKey::RSA.new(File.open("#{ENV["HOME"]}/.ssl/localhost.key").read),
        SSLCertificate: OpenSSL::X509::Certificate.new(File.open("#{ENV["HOME"]}/.ssl/localhost.crt").read),
        SSLCertName: [["CN", WEBrick::Utils::getservername]],
      })
    end
  end
end
```

`bin/rails`をコピーし、`bin/rails-ssl`を作成する。  

```bash
$ cp bin/rails bin/rails-ssl
```

テキストエディタで`bin/rails-ssl`を開き、以下のように変更する。  

```ruby
#!/usr/bin/env ruby
# ↑"ruby.exe"となっている場合があるので修正しておく

# 以下の行を追加する
require_relative '../config/ssl' 

APP_PATH = File.expand_path('../../config/application',  __FILE__)
require_relative '../config/boot'
require 'rails/commands'
```

以下のコマンドでサーバを開始する。  

```bash
$ bundle exec bin/rails-ssl s -b 0.0.0.0
```
