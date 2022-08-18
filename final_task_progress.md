### ゴール
CircleCIで以下の一連のアクションをすべて自動化する
1. CFnによるスタック作成(サーバーのリソース作成)
2. Ansibleによってリソースの構成管理を行う。アプリケーションをデプロイする。
3. Serverspecによるテストを行う


### ゴールまでにやるべきこと
* 構成図を作成する
* CFnテンプレートの作成(OK)
* Ansibleのプレイブックの作成（進行中）
  * git(OK)
  * nginx(OK)
  * rubyインストールのための各パッケージ(OK)
  * rbenv, ruby-build, ruby(OK)
  * Railsアプリのデプロイ
    * ソースコードのclone(OK)
    * MySQLクライアントのインストール&RDSへの接続設定
    * unicorn
* Serverspecのコードを書く(書きたい内容を1日２つ考える。
* CircleCIで１，２，３の各工程をつなぐ。


Serverspecメモ\
* パッケージがインストールされているか\
%w{autoconf bison flex gcc gcc-c++ kernel-devel make m4}.each do |pkg|
  describe package(pkg) do
    it { should be_installed }
  end
end
指定のバージョンか\
describe package('td-agent') do
  it { should be_installed.with_version('1') }
end
