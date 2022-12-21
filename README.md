# システム概要
[ethereumNetStats](https://ethereumnetstats.info/)は、暗号資産イーサリアムの
ネットワークステータスを表示するウェブサイトです。  
このウェブサイトは、典型的なフロントエンドーバックエンドシステムです。  
しかし、バックエンドは単一のサーバーではなく、自宅サーバーとAmazon Lightsailインスタンスとを組み合わせた構成にしています(Fig.1)。
![Fig1](fig1.jpg)
<div style="text-align: center;">Fig.1</div>

## 自宅サーバー
自宅サーバーは、[Geth](https://github.com/ethereum/go-ethereum)、[MySQL](https://www.mysql.com/)、
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

#### データレコーダー  
データレコーダーは、[web3.js](https://github.com/web3/web3.js)、[mysql2](https://github.com/sidorares/node-mysql2#readme)
及び[socket.io](https://socket.io/)を主に使用してTypescriptで製作しています。  

#### ソケットサーバー
ソケットサーバーは、mysql2及びsocket.ioを主に使用してTypescriptで製作しています。

#### ソースコード
私がデータサーバーを構成するために製作したプログラムは以下の通りです。  
これらのプログラムはそれぞれDockerで運用しています。

_データレコーダー_
- [blockDataRecorder]()  
Gethからブロックデータ（ネットワーク上の取引データ、手数料、及び採掘難易度などを示すデータ）を取得してMySQLに記録する。  
- [transactionRecorder]()  
ブロックデータに記録されている取引データの識別子から取引データを取得して、MySQLに記録する。
- [addressRecorder]()  
Gethから取引データを取得し、ネットワーク上で日々増加するアドレスを全てMySQLに記録する。
- [minutelyBasicNetStatsRecorder]()  
ブロックデータに含まれるデータから取引数や各種平均値などを１分ごとに集計してMySQLに記録し、ソケットサーバーに送信する。
- [hourlyBasicNetStatsRecorder]()  
ブロックデータに含まれるデータから取引数や各種平均値などを１時間ごとに集計してMySQLに記録し、ソケットサーバーに送信する。
- [dailyBasicNetStatsRecorder]()  
ブロックデータに含まれるデータから取引数や各種平均値などを１日ごとに集計してMySQLに記録し、ソケットサーバーに送信する。
- [weeklyBasicNetStatsRecorder]()  
ブロックデータに含まれるデータから取引数や各種平均値などを１週間ごとに集計してMySQLに記録し、ソケットサーバーに送信する。

_ソケットサーバー_  
- [socketServer]()  
上記データレコーダーの各種プログラムやMySQLと通信し、各種時間ごとの集計データなどをAmazon Lightsailインスタンスへ送信する。

## Amazon Lightsail

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
