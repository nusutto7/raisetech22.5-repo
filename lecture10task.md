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

<img width= 30% src="https://user-images.githubusercontent.com/75251188/179355269-bf8c3744-a2db-430f-bc98-57a854888f65.png">
