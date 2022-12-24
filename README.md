# ethereumNetStatsについて

<!-- TOC -->
* [システム概要](#システム概要)
  * [ホームサーバー](#ホームサーバー)
    * [Geth](#geth)
    * [MySQL](#mysql)
    * [データサーバー](#データサーバー)
  * [Amazon Lightsailインスタンス](#amazon-lightsailインスタンス)
    * [データプールサーバー](#データプールサーバー)
    * [データパブリッシャー](#データパブリッシャー)
    * [Reactサーバー](#reactサーバー)
* [ソースコード](#ソースコード)
  * [ホームサーバー側](#ホームサーバー側)
    * [データレコーダープログラム](#データレコーダープログラム)
    * [ソケットサーバープログラム](#ソケットサーバープログラム)
  * [Amazon Lightsail側](#amazon-lightsail側)
* [About (English section)](#About)
<!-- TOC -->
# システム概要
[ethereumNetStats](https://ethereumnetstats.info/)は、暗号資産イーサリアムの
ネットワークステータスを表示するウェブサイトです。  
このウェブサイトは、典型的なフロントエンドーバックエンドシステムです。  
しかし、バックエンドは単一のサーバーではなく、ホームサーバーとAmazon Lightsailインスタンスとを組み合わせた構成にしています(Fig.1)。
![Fig1](fig1.jpg)
<div style="text-align: center;">Fig.1</div>

## ホームサーバー
ホームサーバーは、[Geth](https://github.com/ethereum/go-ethereum)、[MySQL](https://www.mysql.com/)、
及びデータサーバーで構成されています(Fig.2)。  
![Fig2](fig2.jpg)
<div style="text-align: center;">Fig.2</div>

### [Geth](https://github.com/ethereum/go-ethereum)
Gethはイーサリアムネットワークに接続するためのクライアントです。

### [MySQL](https://www.mysql.com/)
MySQLはデータベースです。

### データサーバー
データサーバーは、私が[Typescript](https://www.typescriptlang.org)を使用して製作したサーバーで、
データレコーダーとソケットサーバーで構成されています。  
データレコーダーは、Gethから取得したデータの記録や集計を行う複数のプログラムです。  
ソケットサーバーは、データレコーダーによって集計されたデータやAmazon Lightsailインスタンスからの要求に応じて取得したデータの送信を行います。

![Fig3](fig3.jpg)
<div style="text-align: center;">Fig.3</div>

## Amazon Lightsailインスタンス
Amazon Lightsailでは、データプールサーバー、データパブリッシャー、及びReactサーバーの３つのインスタンスを運用しています(Fig.4)。
![Fig4](fig4.jpg)
<div style="text-align: center;">Fig.4</div>

### データプールサーバー
データプールサーバーは、ホームサーバーから受け取ったデータを一時的に保存します。これにより、仮にホームサーバーとの接続が切れたとしても、
データの更新は停止しますが、内部に保存しているデータの提供は続けることができます。  

### データパブリッシャー
データパブリッシャーは、データプールサーバーからデータを受け取ってフロントエンドに配信するためのサーバーです。

### Reactサーバー
Reactサーバーは、[React.js](https://ja.reactjs.org/)を使用して製作したwebサイト[ethereumNetStats](https://ethereumnetstats.info/)を
フロントエンドに提供するサーバーです。このサイトで表示するデータは全てデータパブリッシャーから取得します。  
このサーバーが提供するサイトでは、以下のことが行えます。
- 直近１０ブロック分のブロックデータのリストをリアルタイムで表示
- 各集計期間（１分、１時間、１日、１週間）ごとの 各種集計データのチャート表示
- 全ブロックデータを２５個づつに分割してページングしたリストの表示
- 入力・選択したブロックデータの検索・詳細表示
- 関連のtwitterアカウント[tweether](https://mobile.twitter.com/twe_ether)の最新のタイムラインの表示

# ソースコード
私が製作したプログラムは以下の通りです。

## ホームサーバー側
ホームサーバー側で運用しているプログラムには、上述の通りデータレコーダー用の各種プログラムと、ソケットサーバー用のプログラムがあります。  
これらはそれぞれTypescriptを使用して製作し、Node.jsで実行するDockerコンテナで運用しています。

### データレコーダープログラム  
以下のデータレコーダーのプログラムは、[web3.js](https://github.com/web3/web3.js)、[mysql2](https://github.com/sidorares/node-mysql2#readme)
及び[socket.io](https://socket.io/)が主な使用ライブラリになります。
- [blockDataRecorder](https://github.com/ethereumNetStats/blockDataRecorder)  
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

### ソケットサーバープログラム  
以下のソケットサーバーのプログラムは、mysql2、及びsocket.ioが主な使用ライブラリです。
- [socketServer]()  
上記データレコーダーの各種プログラムやMySQLと通信し、各種時間ごとの集計データやブロックデータの検索結果などをデータプールサーバーへ送信します。

## Amazon Lightsail側
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

# About
<!-- TOC -->
* [Overall system structure](#overall-system-structure)
  * [Home server](#home-server)
    * [Geth](#geth-1)
    * [MySQL](#mysql-1)
    * [Data server](#data-server)
  * [Amazon Lightsail instances](#amazon-lightsail-instances)
    * [Data pool server](#data-pool-server)
    * [Data publisher](#data-publisher)
    * [React server](#react-server)
* [Source code](#source-code)
  * [Home server side](#home-server-side)
    * [Data recorder program](#data-recorder-program)
    * [Socket server program](#socket-server-program)
  * [Amazon Lightsail side](#amazon-lightsail-side)
<!-- TOC -->

# Overall system structure
[ethereumNetStats](https://ethereumnetstats.info/) is a website that displays the network status of the crypto asset ethereum.  
The website is a typical front-end-back-end system.  
However, the backend is not a single server, but a combination of a home server and an Amazon Lightsail instance (Fig. 1).  
![Fig1](fig1.jpg)
<div style="text-align: center;">Fig.1</div>

## Home server
The home server consists of [Geth](https://github.com/ethereum/go-ethereum), [MySQL](https://www.mysql.com/)
and a data server (Fig. 2).  
![Fig2](fig2.jpg)
<div style="text-align: center;">Fig.2</div>

### [Geth](https://github.com/ethereum/go-ethereum)
Geth is a client for connecting to the Ethereum network.  

### [MySQL](https://www.mysql.com/)
MySQL is a database.  

### Data server
The data server is a server I built using [Node.js](https://nodejs.org/ja/) and consists of a data recorder and a socket server.
The data recorder is a set of programs that record and aggregate the data retrieved from Geth.  
The Socket Server is responsible for sending the data acquired by the Data Recorder and 
the data acquired in response to requests from Amazon Lightsail instances.

![Fig3](fig3.jpg)
<div style="text-align: center;">Fig.3</div>

## Amazon Lightsail instances
Three instances of Amazon Lightsail are running: a data pool server, a data publisher, and a React server (Fig. 4).
![Fig4](fig4.jpg)
<div style="text-align: center;">Fig.4</div>

### Data pool server
The data pool server temporarily stores data received from the home server. 
This means that even if the connection to the home server is lost, the Data updates will stop, 
but the internally stored data can continue to be provided.

### Data publisher
The data publisher is a server that receives data from the data pool server and distributes it to the front-end.

### React server
The React server is a server that provides the website [ethereumNetStats](https://ethereumnetstats.info/) produced using [React.js](https://ja.reactjs.org/).
All data displayed on this site is obtained from a data publisher.  
The site provided by this server can do the following
- Display a list of the last 10 blocks data in real time.
- Display charts of various aggregate data for each aggregation period (1 minute, 1 hour, 1 day, 1 week).
- Display of a paged list of all block data divided into 25 pieces each.
- Search and detailed display of input/selected block data.
- Display of the latest timeline of related twitter accounts [tweether](https://mobile.twitter.com/twe_ether).

# Source code
The program I produced is as follows.

## Home server side
The programs running on the home server side include various programs for the data recorder and the socket server, as described above.  
Each of these is produced using Typescript and run in a Docker container running in Node.js.

### Data recorder program
The following data recorder program uses [web3.js](https://github.com/web3/web3.js), [mysql2](https://github.com/sidorares/node-mysql2#readme)
and [socket.io](https://socket.io/) are the main libraries used.
- [blockDataRecorder]()  
  Obtain block data (data indicating transaction data on the network, fees, and mining difficulty, etc.) from the Geth and record it in MySQL.
- [transactionRecorder]()  
  Retrieve the transaction data from the transaction data identifier recorded in the block data and record it in MySQL.
- [addressRecorder]()  
  It retrieves transaction data from Geth and records in MySQL all the addresses that increase daily on the network.
- [minutelyBasicNetStatsRecorder]()  
  The number of transactions and various averages from the data contained in the block data are tabulated every minute, recorded in MySQL, and sent to the socket server.
- [hourlyBasicNetStatsRecorder]()  
  The number of transactions and various average values from the data contained in the block data are aggregated hourly, recorded in MySQL, and sent to the socket server.
- [dailyBasicNetStatsRecorder]()  
  The number of transactions and various average values from the data contained in the block data are aggregated and recorded in MySQL on a daily basis and sent to the socket server.
- [weeklyBasicNetStatsRecorder]()  
  The number of transactions and various averages from the data contained in the block data are compiled and recorded in MySQL on a weekly basis and sent to the socket server.

### Socket server program
The following socket server program uses mysql2 and socket.io as its main libraries.
- [socketServer]()  
  Communicates with the various programs of the above data recorder and MySQL to send various hourly aggregate data, block data search results, etc. to the data pool server.

## Amazon Lightsail side
The programs running on the Amazon Lightsail side include programs for the data pool server, data publisher, and React server, as described above.  
Each of these is produced in Typescript, and a daemon that executes the code transferred to individual instances in Node.js is operated using [forever](https://github.com/foreversd/forever).
- [dataPoolServer]()  
  Communicates with the socket server on the home server side to request, retrieve, and transfer initial data, retrieve and transfer various aggregate data, and transfer user search requests and search results.
- [dataPublisher]()  
  Initial data is retrieved from the data pool server in response to a connection from the front-end and transferred. 
- It also retrieves and transfers various aggregate data sequentially.
  Forwards block data retrieval requests from users to the data pool server and forwards the retrieval results to the front-end.
- [ethereumNetStats]()  
  This is a program that provides the React site described above, in the form of a React program built with React and distributed via [Express](https://expressjs.com/ja/).

<!---
ethereumNetStats/ethereumNetStats is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
