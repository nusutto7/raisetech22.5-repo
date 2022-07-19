★CloudFomationを用いてコード化★

### 事前準備
* Udemyのコースでイメージを掴む
  * https://www.udemy.com/course/aws-cloudformation-master-class/
  * 進捗は50%、理解度は多分10%くらい（手を動かしていないから）
* 第10回講義を視聴
* 【AWS Black Belt Online Seminar】AWS CloudFormation　を視聴
  * https://www.youtube.com/watch?v=Viyqh9fNBjw
  * この動画によれば、スタックは分割するべしとのこと(Network Security Applicationの3層に分けて記述するとよいらしい)
  * (先輩受講生が、これを参考にテンプレートを作成したとのことなので、それを真似させてもらいます)

### 方針
* 2つのAZにEC2、RDSを１つずつを配置し、冗長化構成をとる
* Network Layer, Security Layer, Application Layerの順に、それぞれテンプレートを分割して進める。

### 1. Network Layer(7/16はここまで)
* 構成図は以下のとおり
<img width= 30% src="https://user-images.githubusercontent.com/75251188/179355093-4ee008dd-a85d-4cd2-a4ae-9ccff5f91af9.jpg">

* タイプミスなどでエラーが出たが、ひとまずスタックの作成に成功。
<img width= 30% src="https://user-images.githubusercontent.com/75251188/179355269-bf8c3744-a2db-430f-bc98-57a854888f65.png">

### 2. Security Layer
* 以下のリソースにアタッチするセキュリティグループを、ここでまとめて作成する。設定する必要があるのはIngress(デフォルトだとすべて拒否されてしまうから)。Egressはデフォルトですべて許可なので放置。
* AWS::RDS::DBSecurityGroupの公式ドキュメントを読むと、こんなことが書いてあった。
>Note
>
>DB security groups are a part of the EC2-Classic Platform and as such are not supported in all regions. It is advised to use the AWS::EC2::SecurityGroup resource in those regions instead. 
* …とのことなので、RDS・ALBのセキュリティグループもすべてAWS::EC2::SecurityGroupで定義する。
* 構成図は以下のとおり

!<img width=30% src="https://user-images.githubusercontent.com/75251188/179382123-165f48e5-14b0-4e97-b43b-86daee730a8e.png">

  * EC2
    * SSH接続を許可。port22
    * ALBからのトラフィックを許可。port80
  * RDS
    * EC2からのトラフィックを許可(今回は面倒なので全IP許可)。port3306
  * ALB
    * 全TCP/IPトラフィックを許可。port3000

* 成功
<img width=30% src="https://user-images.githubusercontent.com/75251188/179382138-2a7469b9-69d7-46cc-950a-1d425913039d.png">

### 3. Applicatin Layer
EC2,RDS,ALBをここで構築する。ついでに、キーペアもCFnで一発で作成できるようになったとのことなので、併せてコードを書いてみました。
* 構成図は以下のとおり。
<img width=30% src="https://user-images.githubusercontent.com/75251188/179663350-f7470ef2-180b-428b-8c56-b265ae8df374.jpg">

#### Key Pair

以下を参考に作成。

https://aws.amazon.com/jp/about-aws/whats-new/2022/04/aws-management-features-ec2-key-pairs/
https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-keypair.html


#### EC2
進め方に手間取って時間がかかった。
公式ドキュメントを見て、上から順番に全プロパティの正体を確認。今回の設定に必要ないと判断したプロパティは削っていく。
以下、気になったもののまとめ。
* BlockDeviceMappings
  * インスタンス起動時にアタッチするボリュームの詳細をここに書く
  * ルートボリューム、ボリュームのパーティションの概念など、もしLinuxの基本が分かっていれば、このあたりスムーズに理解できたのではないかと思った。
* ImageId
  * AMIのIDを指定する。"ami-うんたらかんたら"
  * 直接書いてもいいけど、ImageIDはコロコロ変わってしまうらしい。
  * AMIのIDを指定する。直接書いてもいいけど、ImageIDはコロコロ変わってしまうらしい。最新のImageIDはSystem Managerのパラメータストアにて、パブリックパラメータとして提供されているので、それを引用する。
* 他にも共有ホスト・専有ホストの概念とかが出てきた。面白そうだけど理解が追いつかなかったので、課題提出後に調べます。
  
#### RDS
試しにMultiAZ構成をとってみました。こちらもプロパティを片っ端から確認。理解できなかったものはとりあえず削って進める方針で。
基本はCFnの公式ドキュメントとコンソールを対比しながら進める。プロパティの英語の説明が理解できないときは、ユーザーガイド(日本語あり)に頼った。
* ユーザーガイド(https://docs.aws.amazon.com/ja_jp/AmazonRDS/latest/UserGuide/USER_CreateDBInstance.html)
以下、気になったもののまとめ。
* `DeletionPolicy: Delete`とする。デフォルトだとSnapshotになっているので、スタック消す度にスナップショットの作成で時間を取られてしまう。さらにスナップショットは課金アイテムなので、放置はNG。
* DBsecuritygroups と VPCsecuritygroups
  * 前者はVPCの外にあるDBインスタンスへのアクセスを制御するもの。今回はVPC内にDBインスタンスを置いてるので後者の設定が必要で、前者はいらない。言葉の見てくれだけだと勘違いしてしまう。
* MultiAZ化
  * AvailabilityZoneプロパティは設定しない。設定すると多分怒られる。
  * AWS::RDS::DBSubnetGroupで複数のAZを指定してあげればOK(のはず)。
* その他プロパティは成り行きに任せる。適宜パラメータ化。


#### ALB
知識が弱かったのが原因でとても苦戦した。
タイプミスも多く、Applicationテンプレート内でRDSなどと一緒にしてしまうと、スタックの削除にものすごく時間がかかってしまうため、これだけテンプレートを独立させることにした。
ユーザーからのリクエストを受けるポート、ヘルスチェックを発信するポート、ヘルスチェックの宛先ポート、ヘルスチェックの応答を受けるポート。
この4つを、正しいリソースの正しいプロパティでそれぞれ設定してあげる必要がある。

#### S3
特になし。

### 4. 心掛けたこと
* 分からないことがあったら、とりあえずQiitaなど、読みやすい、理解しやすい記事にどんどん当たる。ただし最後に必ず公式ドキュメントに当たる。
  * 公式ドキュメントが英語で読めるようになるのがきっと最強。日本語ドキュメントは完備されておらず、英語の直訳っぽい(AWS語?)。英語で読めるようになってしまうのが一番てっとり早い気がした。
  * あと、英文読める力は他のところでも活かせるんじゃないかと思った。
  * すごく疲れた。

３３
