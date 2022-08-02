### CI/CDについて

* CI = Continuous Integration　
テストの自動化。リリース可能な状態を保つ。
* CD = Continuous Delivery
リリースまで自動化。

### 仕組み
* GithubのリポジトリとCircleCIを連携しておく
* 連携先リポジトリに変更が生じると、CircleCIがそれを検知
* CircleCIは`config.yml`を読み込んで実行
* `jobs`にひとまとまりのテスト手順を書く。`workflowｓ`に実行したいjobsの流れを書く。

今回の課題ではここまでの流れを掴む。
自動デプロイまで行うとなると、EC2にログインするための認証情報の設定、IAMロールの割り当て、デプロイ作業のコード化などの工程が生じる、はず。
(そのあたりは次週以降の課題???)

### やってみた
第10回の課題で作成したCFnのテンプレートファイルをテスト対象として使う。
mainブランチに`.circleci/config.yml`を作成。サンプルコンフィグの内容を貼り付ける。
今回の課題用にブランチを切り、ALBのテンプレートファイルをアップロードしてみる。
<img width=40% alt="スクリーンショット 2022-08-02 19 15 56" src="https://user-images.githubusercontent.com/75251188/182384665-9088677a-49fd-43ff-a6d0-1df24e3c9c65.png">


続いてNetwork_layerのテンプレートをアップロード。

<img width=40% alt="スクリーンショット 2022-08-02 19 18 34" src="https://user-images.githubusercontent.com/75251188/182389519-f19704e2-75f2-4a38-857e-ebe7816c59fe.png">

サブネットのAZをハードコーディングするなとのこと。これはびっくり。
そこでAZのPropertiesを以下のように修正。
```:yml
Properties: 
      AvailabilityZone: !Select
        - 0
        - Fn::GetAZs: !Ref AWS::Region
```
自動でテストが入る。今度は成功。

<img width=40% alt="スクリーンショット 2022-08-02 19 49 02" src="https://user-images.githubusercontent.com/75251188/182390798-742e584b-99ad-4b24-a43d-9a52f497c47f.png">


### ローカルCLIでのconfig.ymlのバリデーション
公式の手順に沿ってローカルCLIをインストール(https://circleci.com/docs/ja/local-cli)

`.circleci/config.yml`のバリデーションを実施する。直に`config.yml`を置いてもチェックしてくれない。
ディレクトリ構成をこの通りにしてあげる必要がある。
まず、`steps`というキー名をわざと間違えてバリデーションを実施。

```:terminal
$ circleci config validate
Error: ERROR IN CONFIG FILE:
[#/jobs/cfn-lint] 0 subschemas matched instead of one
1. [#/jobs/cfn-lint] only 1 subschema matches out of 2
|   1. [#/jobs/cfn-lint] 2 schema violations found
|   |   1. [#/jobs/cfn-lint] extraneous key [steeeeeeeps] is not permitted
|   |   |   Permitted keys:
|   |   |     - description
|   |   |     - parallelism
|   |   |     - macos
|   |   |     - resource_class
|   |   |     - docker
|   |   |     - steps
|   |   |     - working_directory
|   |   |     - circleci_ip_ranges
|   |   |     - machine
|   |   |     - environment
|   |   |     - executor
|   |   |     - shell
|   |   |     - parameters
|   |   |   Passed keys:
|   |   |     - executor
|   |   |     - steeeeeeeps
|   |   2. [#/jobs/cfn-lint] required key [steps] not found
2. [#/jobs/cfn-lint] expected type: String, found: Mapping
|   Job may be a string reference to another job
```

キー名を`steps`に修正し、再度バリデーション実行。OK。
```
$ circleci config validate
Config file at .circleci/config.yml is valid.
```
あくまでもチェックしてくれるのは文法的な間違いまでなので、過信は禁物?

### 感じたこと・学んだこと
こうなると自前のアプリが欲しくなる。
メンテナンスして、テストして…を繰り返しながらアプリを改良していく作業は面白そう。
