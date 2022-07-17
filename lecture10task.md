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
* とのことなので、RDS・ALBのセキュリティグループもすべてAWS::EC2::SecurityGroupで定義する。

  * EC2
    * SSH接続を許可。port22
    * ALBからのトラフィックを許可。port80
  * RDS
    * EC2からのトラフィックを許可(今回は面倒なので全IP許可)。port3306
  * ALB
    * 全TCP/IPトラフィックを許可。port３０００
