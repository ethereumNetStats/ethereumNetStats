[English](#Overall-system-structure)

# システム概要
[ethereumNetStats](https://ethereumnetstats.info/)は、暗号資産イーサリアムの
ネットワークステータスを表示するウェブサイトです。  
このウェブサイトは、典型的なフロントエンドーバックエンドシステムです。  
しかし、バックエンドは単一のサーバーではなく、自宅サーバーとAmazon Lightsailインスタンスとを組み合わせた構成にしています(Fig.1)。
![Fig1](fig1.jpg)
<div style="text-align: center;">Fig.1</div>

## ホームサーバー
ホームサーバーは、[Geth](https://github.com/ethereum/go-ethereum)、[MySQL](https://www.mysql.com/)、
及びデータサーバーで構成されています(Fig.2)。  
Gethはイーサリアムネットワークに接続するためのクライアント、MySQLはデータベースでそれぞれ公式サイトやDockerHubなどで配布されています。  
データサーバーは、私が[Node.js](https://nodejs.org/ja/)を使用して製作したサーバーです。  
![Fig2](fig2.jpg)
<div style="text-align: center;">Fig.2</div>

### データサーバー
データサーバーは、データレコーダーとソケットサーバーで構成されています。  
データレコーダーは、Gethから取得したデータの記録や集計を行う複数のプログラムです。  
ソケットサーバーは、データレコーダーによって集計されたデータやAmazon Lightsailインスタンスからの要求に応じて
取得したデータの送信を行います。

![Fig3](fig3.jpg)
<div style="text-align: center;">Fig.3</div>

## Amazon Lightsailインスタンス
Amazon Lightsailでは、データプールサーバー、データパブリッシャー、及びReactサーバーの３つのインスタンスを運用しています(Fig.4)。  
データプールサーバーは、ホームサーバーから受け取ったデータを一時的に保存します。これにより、仮にホームサーバーとの接続が切れたとしても、
データの更新は停止しますが、内部に保存しているデータは提供を続けることができます。  
データパブリッシャーは、データプールサーバーからデータを受け取ってフロントエンドに配信するためのサーバーです。  
Reactサーバーは、[React.js](https://ja.reactjs.org/)を使用して製作したwebサイト[ethereumNetStats](https://ethereumnetstats.info/)を
フロントエンドに提供するサーバーです。このサイトで表示するデータは全てデータパブリッシャーから取得します。  
![Fig4](fig4.jpg)
<div style="text-align: center;">Fig.4</div>

### Reactサーバー
このサーバーが提供するサイトでは、以下のことが行えます。
- 直近１０ブロック分のブロックデータのリスト表示
- 各集計期間（１分、１時間、１日、１週間）ごとの 各種集計データのチャート表示
- 全ブロックデータを２５個づつに分割してページングしたリストの表示
- 入力・選択したブロックデータの検索・詳細表示
- 関連のtwitterアカウント[tweether](https://mobile.twitter.com/twe_ether)の最新のタイムラインの表示

## ソースコード
私が製作したプログラムは以下の通りです。

### ホームサーバー側
ホームサーバー側で運用しているプログラムには、上述の通りデータレコーダー用の各種プログラムと、ソケットサーバー用のプログラムがあります。  
これらはそれぞれTypescriptを使用して製作し、Node.jsで実行するDockerコンテナで運用しています。

#### データレコーダープログラム  
以下のデータレコーダーのプログラムは、[web3.js](https://github.com/web3/web3.js)、[mysql2](https://github.com/sidorares/node-mysql2#readme)
及び[socket.io](https://socket.io/)が主な使用ライブラリになります。
- [blockDataRecorder]()  
Gethからブロックデータ（ネットワーク上の取引データ、手数料、及び採掘難易度などを示すデータ）を取得してMySQLに記録します。  
- [transactionRecorder]()  
ブロックデータに記録されている取引データの識別子から取引データを取得して、MySQLに記録します。
- [addressRecorder]()  
Gethから取引データを取得し、ネットワーク上で日々増加するアドレスを全てMySQLに記録します。
- [minutelyBasicNetStatsRecorder]()  
ブロックデータに含まれるデータから取引数や各種平均値などを１分ごとに集計してMySQLに記録し、ソケットサーバーに送信します。
- [hourlyBasicNetStatsRecorder]()  
ブロックデータに含まれるデータから取引数や各種平均値などを１時間ごとに集計してMySQLに記録し、ソケットサーバーに送信します。
- [dailyBasicNetStatsRecorder]()  
ブロックデータに含まれるデータから取引数や各種平均値などを１日ごとに集計してMySQLに記録し、ソケットサーバーに送信します。
- [weeklyBasicNetStatsRecorder]()  
ブロックデータに含まれるデータから取引数や各種平均値などを１週間ごとに集計してMySQLに記録し、ソケットサーバーに送信します。

#### ソケットサーバープログラム  
以下のソケットサーバーのプログラムは、mysql2、及びsocket.ioが主な使用ライブラリです。
- [socketServer]()  
上記データレコーダーの各種プログラムやMySQLと通信し、各種時間ごとの集計データやブロックデータの検索結果などをデータプールサーバーへ送信します。

### Amazon Lightsail側
Amazon Lightsail側で運用しているプログラムには、上述の通りデータプールサーバー、データパブリッシャー、Reactサーバー用のプログラムがあります。  
これらはそれぞれTypescriptで製作し、個別のインスタンスに転送したコードをNode.jsで実行するデーモンを[forever](https://github.com/foreversd/forever)を使用して
運用しています。
- [dataPoolServer]()  
ホームサーバー側のソケットサーバーと通信して、初期データの要求・取得・転送、各種集計データの取得・転送、及びユーザーの検索要求の転送、検索結果の転送をします。  
- [dataPublisher]()  
フロントエンドからの接続に応じて初期データをデータプールサーバーから取得し転送します。また、各種集計データを逐次取得・転送します。
ユーザーからのブロックデータの検索要求をデータプールサーバーに転送し、検索結果をフロントエンドに転送します。
- [ethereumNetStats]()  
上述したReactサイトを提供するプログラムです。Reactで製作しビルドしたものを[Express](https://expressjs.com/ja/)で配布する形式です。


# Overall system structure
The ethereumNetStats project is to build a website to display network status of the ethereum.  
The website is composed of typically front-end and back-end system.  
However, the back-end is not a single server. It is composed of the Amazon Lightsail instances and my home server like below.  

![Fig1](fig1.jpg)
<div style="text-align: center;">
Fig.1
</div>

# Home server  
The home server is composed of the [Geth](https://github.com/ethereum/go-ethereum), the [MySQL](https://www.mysql.com/), 
and the Data server as shown in fig.2.
![Fig2](fig2.jpg)

## Data server  
My job inside the Home server is to build the Data server.  

<!---
ethereumNetStats/ethereumNetStats is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
