# HAP(Hatohol Arm Plugin)

## ToDO

Initiation
serverTypeのURL
エラーコード
AA書く

## 概要

Hatohol Arm Plugin(以下，HAP)とは，Hatoholが定義しているデータ（イベント，トリガー，アイテム，etc...）をHatohol Arm Plugin Interfaceという概念，インターフェースにのっとり，Hatoholサーバーとデータのやりとりを行うことが出来るプラグインの総称です。このドキュメントでは，HAPを作成するにあたっての必要な情報をまとめています。
不明点についてはHatoholコミュニティーにお問い合わせください。[hatohol-users@sourceforge.net]

HAPでは通信プロトコルにJSON-RPC 2.0を採用しています。JSON-RPCの仕様については[公式リファレンス](http://www.jsonrpc.org/specification)をお読みください。

![overview](hapi_overview.png)

## Hatohol動作全容

それっぽいAA書く

## 注意事項

 - 現在，Hatoholはutf-8を標準的な文字コードとして，作成されています。Hatoholサーバーに送信するデータの文字コードにはutf-8を使用することを推奨します。
 - リクエスト・レスポンスで使用するIDオブジェクトの値には，十分なランダム性を必要とします。
 - JSON-RPCにはバッチリクエストという，複数のリクエストを同時に送信する文法が存在しますが，Hatoholはこの文法を用いたリクエストには対応していません。
 - オブジェクトの名前として各要素のID，値としてそのIDに対応している各値を更にオブジェクト形式で記述してください。ここに記述するIDはユニークである必要があります。
 - 各コマンド解説においてMandatoryがYesとなっている要素を使用しない場合は，値として「""」を入力しリクエストやレスポンスを送信してください。

## データ型解説

|名前|JSON型|解説|
|:---|:---------|:---|
|timestamp|string|時刻フォーマットはyyyyMMDDHHmmss.nnnnnnnnnです。小数点以下の時刻については省くことが出来ます。また，小数点以下には9桁までしか値を挿入することはできません。小数点以下を省いた場合，または小数点以下が9桁未満の場合には余った桁部に0が挿入されます。(Ex.100 -> 100.000000000, 100.1234 -> 100.123400000)。|
|boolean|true, false|true or falseを指定し，その値の真偽を示します|

## Initiationについて

応相談

## コマンド解説

 - 全てのコマンドには想定される標準的な動作が存在しますが，全て任意のタイミングで使用することができます。
 - 各コマンドにMandatory項目を記載しています。この項目がYesのコマンドは実装を省略することができません。
 - MandatoryにYesがついていないコマンドは実装を省略することが可能ですが，req~~~~コマンドのようにHatoholサーバーからリクエストを受けるコマンドの実装を省略している場合は，そのコマンドを実装していないことをHatoholサーバーに示す必要があります。errorオブジェクトにて◯◯エラーを返すように実装してください。[[エラー一覧](#user-content-errorCode)]
 - HAP -> SRV,またSRV -> HAPカラムは各コマンドが送られる通信の方向を表しています。例えばHAP -> SRVがYesとなっているコマンドは，プラグインがHatoholサーバーにコマンドを送信することを表します。
 - 通信の方向がHAP -> SRVであるコマンド内のnumber型範囲は0~2147483647です。
 - send~~~コマンドは，送信したデータのデータベース書き込み成否をresultオブジェクトとして受け取ります。結果の値については[[一覧](#user-content-result)]をご覧ください。

|コマンド名               |解説|HAP -> SRV|SRV -> HAP|メソッドタイプ|Mandatory|
|:------------------------|:---|:--------:|:--------:|:------------:|:-------:|
|[getMonitoringServerInfo](#user-content-getMonitoringServerInfo)|監視サーバーの接続情報やポーリング間隔等を取得します|Yes|-|method|Yes|
|[getLastInfo](#user-content-getLastInfo)|Hatoholサーバーに保存されている指定した要素の最新情報を取得します|Yes|-|method|-|
|[sendHosts](#user-content-sendHosts)|監視サーバーが監視しているホスト一覧をHatoholサーバーに送信します|Yes|-|method|-|
|[sendHostGroups](#user-content-sendHostGroups)|ホストグループの情報をHatoholサーバーに送信します|Yes|-|method|-|
|[sendHostGroupElements](#user-content-sendHostGroupElements)|ホストのホストグループ所属情報をHatoholサーバーに送信します|Yes|-|method|-|
|[sendTriggers](#user-content-sendTrigges)|トリガーをHatoholサーバーに送信します。送信するトリガーはオプションで指定することが出来ます|Yes|-|method|-|
|[sendEvents](#user-content-sendEvents)|アップデートされたイベントをHatoholサーバーに送信します|Yes|-|method|-|
|[sendVirtualRelations](#user-content-sendVirtualRelations)|ホスト同士のVM親子関係をHatoholサーバーに送信します|Yes|-|method|-|
|[sendArmInfo](#user-content-sendArmInfo)|HAPの接続情報をHatoholサーバーに送信します|Yes|-|method|-|
|[reqFetchItems](#user-content-reqFetchItems)|Hatoholサーバーが全てのトリガーを要求しているときにHAPに送信されます|-|Yes|method|-|
|[reqFetchHistory](#user-content-reqFetchHistory)|Hatoholサーバーがヒストリーを要求しているときにHAPに送信されます|-|Yes|method|-|
|[reqFetchTriggers](#user-content-reqFetchTriggers)|Hatoholサーバーが全てのトリガーを要求しているときにHAPに送信されます|-|Yes|method|-|
|[reqTerminate](#user-content-reqTerminate)|HAPとHatoholサーバーとの接続を終了させます|-|Yes|notification|Yes|

### getMonitoringServerInfo(method)

ポーリング時間毎にHatoholサーバーに自身の監視サーバー情報を問い合わせることを標準的な動作としますが，任意のタイミングで問い合わせることもできます。

***params***

getMonitoringServerInfoメソッドには引数が存在しません。nullオブジェクトとしてparamsを送信してください。

***result***

|名前|型|Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:----------:|:---------:|:------:|:---|
|serverId          |number|Yes|-|正の整数|監視サーバーのserverId|
|url               |string|Yes|-|-|監視サーバーのURL [[解説](#user-content-serverType)]|
|type              |string|Yes|-|-|監視サーバーの種類 [[一覧](#user-content-serverType)]|
|nickName          |string|Yes|-|65535byte以内|監視サーバーのニックネーム|
|userName          |string|Yes|-|65535byte以内|監視サーバーのユーザーネーム|
|password          |string|Yes|-|65535byte以内|監視サーバーのパスワード|
|dbName            |string|Yes|-|65535byte以内|監視対象のデータベースのパスワード|
|pollingIntervalSec|number|Yes|-|正の整数|ポーリングを行う間隔|
|retryIntervalSec  |number|Yes|-|正の整数|ポーリングが失敗した場合，リトライを行うまでの間隔|
|extra             |string|Yes|-|65535byte以内|プラグイン固有の情報を格納することができる|

```
{"jsonrpc":"2.0", "result":{"hostName":"exampleHost", "type":0, "ipAddress":"127.0.0.1", "nickName":"exampleName", "userName":"Admin", "password":"examplePass", "dbName":"", "port":80, "pollingIntervalSec":30, "retryIntervalSec":10}, "id":1}
```

### getLastInfo(method)

send~~~コマンドでHatoholサーバーに送信，保存されたlastInfo情報を呼び出します。
また，初回起動時など，HatoholサーバーにlastInfoが保存されていない場合，resultオブジェクトはnullとして返ってきます。
send~~~コマンドでは，このコマンドで取得したlastInfoを用いて，前回送信したデータと現在所持しているデータの差分データをHatoholサーバーに送信することができます。

***params***

```
{"jsonrpc":"2.0", "method":"getLastInfo", "params":"trigger", "id":1}
```

|paramsの値|
|:---------|
|"trigger"|
|"event"  |
|"host"|
|"hostGroup"|
|"hostGroupElement"|
|"relation"|

***result***

|名前|型 |Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:-------:|:----------:|:------:|:---|
|result|string|Yes|-|65535byte以内|Hatoholサーバーに保存されている指定した要素の最新情報|

```
{"jsonrpc":"2.0", "result":"201504011349", "id":1}

この例ではlastInfoとしてtimestampが返ってきています

```

### sendHosts(method)

Hatoholサーバーとの接続完了時，またはHAPが内部的に保存している登録ホスト情報が変更された際に全てのホスト情報をHatoholサーバーに送信することを標準的な動作とします。

[getLastInfo](#user-content-getLastInfo)を用いて取得，またはHAPプロセス自身が保管している最新ホスト情報を基に，その時点から現時点までに追加されたホストをHatoholサーバーに送信するか，または全てのホストを送信します。

***params***

名前：hosts, lastInfo, option

各オブジェクトの値：

|名前|型|Mandatory|デフォルト値|値の範囲|解説|
|:--|:--|:--|:--|:--|:--|
|hosts|object|Yes|-|-|ホスト情報を格納するオブジェクトを配置します。詳細は次のテーブルを確認してください|
|option|string|Yes|-|-|送信オプション[[一覧]("user-content-option")]の中から状況に応じた送信オプションを選択してください|
|lastInfo|string|No|null|65535byte以内|最後に送信したホストグループの情報を送信する。この情報が[getLastInfo](#user-content-getLastInfo)の返り値になる|

***hostオブジェクト***

オブジェクトの名前：ホストID

オブジェクトの値：

|名前|型 |Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:-------:|:----------:|:------:|:---|
|hostName|string|Yes|-|65535byte以内|監視サーバーが監視しているホスト名|

```
{"jsonrpc":"2.0","method":"sendHosts", "params":{"hosts":{1":"exampleHostName1"},"option":"UPDATE","lastInfo":"201504091052"}, "id":1}
```

***result***

```
{"jsonrpc":"2.0", "result":"SUCCESS", "id":1}
```

### sendHostGroups(method)

[getLastInfo](#user-content-getLastInfo)を用いて取得，またはHAPプロセス自身が保管している最新ホストグループ情報を基に，その時点から現時点までに追加されたホストグループをHatoholサーバーに送信するか，または全てのホストグループを送信します。

***params***

名前：hostGroups, lastInfo, option

各オブジェクトの値：

|名前|型|Mandatory|デフォルト値|値の範囲|解説|
|:--|:--|:--|:--|:--|:--|
|hostGroups|object|Yes|-|-|ホストグループ情報を格納するオブジェクトを配置します。詳細は次のテーブルを確認してください|
|option|string|Yes|-|-|送信オプション[[一覧]("user-content-option")]の中から状況に応じた送信オプションを選択してください|
|lastInfo|string|No|null|65535byte以内|最後に送信したホストグループの情報を送信する。この情報が[getLastInfo](#user-content-getLastInfo)の返り値になる|

***hostGroupsオブジェクト***

オブジェクトの名前：グループID

オブジェクトの値：

|名前|型 |Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:-------:|:----------:|:------:|:---|
|groupName|string|Yes|-|65535byte以内|監視ホストが所属するホストグループ|

```
{"jsonrpc":"2.0","method":"sendHostGroups", "params":{"hostGroups":{"1":"Group1", "2":"Group2"},"option":"ALL", "lastInfo":"201504091049"}, "id":1}
```

***result***

```
{"jsonrpc":"2.0", "result":"SUCCESS", "id":1}
```

### sendHostGroupElements(method)

ホストのグループ所属情報をHatoholサーバーに送信します。[sendHostsGroup](#user-content-sendHostGroups)と合わせて使用することを標準的な動作とします。

[getLastInfo](#user-content-getLastInfo)を用いて取得，またはHAPプロセス自身が保管している最新ホストグループ所属情報を基に，その時点から現時点までに追加されたホストグループ所属情報をHatoholサーバーに送信するか，または全てのホストグループ所属情報を送信します。

***params***

名前：hostGroupElements, lastInfo, option

各オブジェクトの値：

|名前|型|Mandatory|デフォルト値|値の範囲|解説|
|:--|:--|:--|:--|:--|:--|
|hostGroupElements|object|Yes|-|-|ホストグループ所属情報を格納するオブジェクトを配置します。詳細は次のテーブルを確認してください|
|option|string|Yes|-|-|送信オプション[[一覧]("user-content-option")]の中から状況に応じた送信オプションを選択してください|
|lastInfo|string|No|null|65535byte以内|最後に送信したホストグループ所属情報の情報を送信する。この情報が[getLastInfo](#user-content-getLastInfo)の返り値になる|

***hostオブジェクト***

オブジェクトの名前：ホストID

オブジェクトの値：

|名前|型 |Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:-------:|:----------:|:------:|:---|
|groupId|number|Yes|- |正の整数|監視ホストが所属しているグループのID|

```
{"jsonrpc":"2.0","method":"sendHostGroupElements", "params":{"hostGroupsElements":{"1":1, "2":2},"lastInfo":"201504091056","option":"ALL"}, "id":1}
```

***result***

```
{"jsonrpc":"2.0", "result":"SUCCESS", "id":1}
```

### sendTriggers(method)

[getLastInfo](#user-content-getLastInfo)を用いて取得，またはHAPプロセス自身が保管している最新トリガー情報を基に，そのトリガーから現時点までに更新されたトリガーをHatoholサーバーに送信するか，全てのトリガーを送信します。

***params***

名前：triggers, lastInfo, option

各オブジェクトの値：

|名前|型|Mandatory|デフォルト値|値の範囲|解説|
|:--|:--|:--|:--|:--|:--|
|triggers|object|Yes|-|-|トリガー情報を格納するオブジェクトを配置します。詳細は次のテーブルを確認してください|
|option|string|Yes|-|-|送信オプション[[一覧]("user-content-option")]の中から状況に応じた送信オプションを選択してください|
|lastInfo|string|No|null|65535byte以内|最新トリガーの情報を送信する。この情報が[getLastInfo](#user-content-getLastInfo)の返り値になる|

***triggersオブジェクト***

HAP自身が所持しているトリガーIDとホストIDを数字ではなく，"_SELF_"と記述し送信することで，Hatoholサーバーが送信されたトリガーをSELFトリガーと判別します。

|名前|型 |Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:-------:|:----------:|:------:|:---|
|status        |string|Yes|-|-            |トリガーのステータス [[一覧](#user-content-triggerStatus)]|
|severity      |string|Yes|-|-            |トリガーの種別 [[一覧](#user-content-triggerSeverity)]|
|lastChangeTime|string|Yes|-|65535byte以内|トリガーが最後に更新された時間|
|hostId        |string|Yes|-|正の整数     |トリガーが所属するホストID|
|hostName      |string|Yes|-|65535byte以内|トリガーが所属するサーバーのホスト名|
|brief         |string|Yes|-|65535byte以内|トリガーの概要|
|extendedInfo  |string|Yes|-|65535byte以内|上記の情報以外の必要な情報。主にWebUI上にデータを表示する際に用いられる|

```
{"jsonrpc":"2.0", "method":"sendTriggers", "params":{"option":"UPDATED", "lastInfo":"201504061606", "Triggers":{"1":{"status":"OK", "severity":"INFO","lastChangeTime":"201503231758", "hostId":"1", "hostName":"exampleName", "brief":"example brief", "extendedInfo": "sample extended info"}, "_SELF_":{"status":"OK", "severity":"INFO","lastChangeTime":"201503231760", "hostId":"_SELF_ "hostName":"exampleName", "brief":"example brief", "extendedInfo": "sample extended info"}}},"id":1}
```

***result***

```
{"jsonrpc":"2.0", "result":"SUCCESS", "id":1}
```

### sendEvents(method)

[getLastInfo](#user-content-getLastInfo)を用いて取得，またはHAPプロセス自身が保管している最新イベント情報を基に，そのイベントから現時点までに発生したイベントをHatoholサーバーに送信します。

***params***

オブジェクトの名前：events, lastInfo

各オブジェクトの値：

|名前|型|Mandatory|デフォルト値|値の範囲|解説|
|:--|:--|:--|:--|:--|:--|
|events|object|Yes|-|-|イベント情報を格納するオブジェクトを配置します。詳細は次のテーブルを確認してください。|
|lastInfo|string|No|null|65535byte以内|最新イベントの情報を送信する。この情報が[getLastInfo](#user-content-getLastInfo)の返り値になる|

***eventsオブジェクト***

オブジェクトの名前：イベントID

オブジェクトの値：

|名前|型 |Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:-------:|:----------:|:------:|:---|
|time        |string|Yes|-|65535byte以内|イベントが発生した時刻|
|type        |string|Yes|-|-|イベントのタイプ [[一覧](#user-content-eventType)]|
|triggerId   |number|Yes|-|正の整数     |このイベントを発火させたトリガーID|
|status      |string|Yes|-|-|トリガーのステータス [[一覧](#user-content-triggerStatus)]|
|severity    |string|Yes|-|-|トリガーの種別 [[一覧](#user-content-triggerSeverity)]|
|hostId      |string|Yes|-|65535byte以内|イベントが発生したホストのID|
|hostName    |string|Yes|-|65535byte以内|イベントが発生したホストの名前|
|brief       |string|Yes|-|65535byte以内|イベントの説明。Web上に表示される情報|
|extendedInfo|string|Yes|-|65535byte以内|briefには書いていない追加の情報を記述できます|

```
{"jsonrpc":"2.0", "method":"sendUpdatedEvents", "params":{"events":{"1":{"time":"201503231513, "type":"GOOD", "triggerId":2, "status": "OK","severity":"INFO":, "hostId":3, "hostName":"exampleName", "brief":"example brief", "extendedInfo": "sampel extended info"}}, "lastInfo":"201504011759"},"id":1}
```

***result***

```
{"jsonrpc":"2.0", "result":"SUCCESS", "id":1}
```

### sendVirtualRelations(method)

指定したホストのペアがVMマシンの親子関係であることをHatoholサーバーに送信します。現在は複数のハイパーバイザが存在することを想定していないため，子のホストIDはユニークである必要があります。

***params***

オブジェクトの名前：子ホストID
オブジェクトの値：親ホストID

```
{"jsonrpc":"2.0", "method":"sendVirtualRelations", "params":{"12":"10", "11":"10"},"id":1}
```

***result***

```
{"jsonrpc":"2.0", "result":"SUCCESS", "id":1}
```

### sendArmInfo(method)

HostやTrigger，Event情報の送信処理が行われるたびにHatoholサーバーに送信することを標準的な動作としますが，任意に送信してもかまいません。最小間隔は１秒（MUST），最大間隔はgetMonitoringServerInfoで取得したポーリング時間の2倍（SHOULD）とする。

```
SRV                             HAP
 |                               |
 |<---------Host-----------------|
 |                               |
 |<---------Trigger--------------|
 |                               |
 |<---------Event----------------|
 |                               |
 |<---------ArmInfo--------------|
 |                           |   |
 |                           |   |
 |                  polling sec  |
 |                           |   |
 |<---------Host-----------------|
```

***params***

|名前|型|Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:---------:|:----------:|:------:|:---|
|lastStatus         |string   |Yes|-|-|最新のポーリング結果 [[一覧](#user-content-armInfoStatus)]|
|failureReason      |string   |Yes|-|65535byte以内|情報取得が失敗した理由|
|lastSuccessTime    |timestamp|Yes|-|-|最後に情報取得が成功した時刻|
|lastFailureTime    |timestamp|Yes|-|-|最後に情報取得が失敗した時刻|
|numSuccess         |number   |Yes|-|正の整数|HAPが起動してから情報取得に成功した回数|
|numFailure         |number   |Yes|-|正の整数|HAPが起動してから情報取得に失敗した回数|

```
{"jsonrpc":"2.0", "method":"sendArmInfo", "params":{"lastStatus":"INIT", "failureReason":"Example reason", "lastSuccessTime":"201503131611", "lastFailureTime":"201503131615", "numSuccess":165, "numFailure":10}, "id":1}
```

***result***

```
{"jsonrpc":"2.0", "result":"SUCCESS", "id":1}
```

### reqFetchItems(method)

このコマンドは、Hatoholサーバーがアイテムを要求しているときにHAPに送信されます。HAPはレスポンスとしてアイテムIDをオブジェクトの名前，アイテム値をオブジェクトの値とした配列にして全てのアイテムを返す必要があります。

***params***

reqFetchItemsメソッドには引数が存在しません。paramsをnullオブジェクトにして送信してください。

***result***

|名前|型|Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:----------:|:---------:|:------:|:---|
|serverId     |number|Yes|-|正の整数     |アイテムが所属するサーバーのID|
|hostId       |number|Yes|-|正の整数     |アイテムが所属するホストのID|
|brief        |string|Yes|-|65535byte以内|各アイテムの概要|
|lastValueTime|string|Yes|-|65535byte以内|各アイテムが最後に更新された時間|
|lastValue    |string|Yes|-|65535byte以内|各アイテムが最後に更新された際の値|
|prevValue    |string|Yes|-|65535byte以内|各アイテムが最後に更新される前の値|
|itemGroupName|string|Yes|-|65535byte以内|アイテムをグループ分けしたもの<br>任意のグループ名をご使用ください|
|unit         |string|Yes|-|65535byte以内|valueの単位|

```
{"jsonrpc:"2.0", "result":{"serverId":1, "hostId":"1", "brief":"example brief", "lastValueTime":"201503231045","lastValue":"example value", "prevValue": "example previous value", "itemGroupName":"example group", "unit",""}}
```

### reqFetchHistory(method)

このコマンドは、Hatoholサーバーがヒストリーを要求しているときにHAPに送信されます。HAPはレスポンスとして，指定時間の範囲に当てはまるヒストリーのアイテムIDをオブジェクトの名前，アイテム値をオブジェクトの値とした配列を返す必要があります。


***params***

|名前|型 |Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:-------:|:----------:|:------:|:---|
|hostId   |string|Yes|-|255文字以内|ヒストリーのアイテムが所属しているホストID|
|itemId   |number|Yes|-|正の整数   |ヒストリーのアイテムID|
|valueType|string|Yes|-|-|取得するヒストリーの値の型 [[一覧](#user-content-itemValueType)]|
|beginTime|string|Yes|-|255文字以内|ヒストリー取得域の始点時間を指定します|
|endTime  |string|Yes|-|255文字以内|ヒストリー取得域の終点時間を指定します|

```
{"jsonrpc":"2.0", "method":"reqFetchHistory", "params":{"hostId":"1", "itemId":1, "valueType":"INTERGER", "beginTime":"201503231513, "beginTime":"201503231513"}},"id":1}
```

***result***

オブジェクトの名前:アイテムID

オブジェクトの値:

|名前|型 |Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:-------:|:----------:|:------:|:---|
|value |string|Yes|-|65535byte以内|clock時点でのアイテムの値|
|clock |string|Yes|-|65535byte以内|このヒストリーの値が記録された時刻|

```
{"jsonrpc:"2.0", "result":{"1":[{"value":"exampleValue","clock":"201503231130"},{"value":"exampleValue2","clock":"201503231130"}]}}
```

### reqFetchTriggers(method)

このコマンドは、Hatoholサーバーが全てのトリガーを要求しているときにHAPに送信されます。HAPはレスポンスとして全てのトリガーの配列を返す必要があります。

***params***

reqFetchTriggersメソッドには引数が存在しません。paramsをnullオブジェクトにして送信してください。

***result***

|名前         |型|Mandatory|デフォルト値|値の範囲|解説|
|:------------|:----|:----:|:----------:|:------:|:---|
|status        |number|Yes|-|正の整数     |トリガーのステータス|
|severity      |string|Yes|-|-|トリガーの種別 [[一覧](#user-content-triggerSeverity)]|
|lastChangeTime|string|Yes|-|65535byte以内|トリガーが最後に更新された時間|
|hostId        |number|Yes|-|正の整数     |監視サーバー内で設定されているホストID|
|hostName      |string|Yes|-|65535byte以内|トリガーが所属するサーバーのホスト名|
|brief         |string|Yes|-|65535byte以内|トリガーの概要|
|extendedInfo  |string|Yes|-|65535byte以内|上記の情報以外の必要な情報。主にWebUI上にデータを表示する際に用いられる|

```
{"jsonrpc":"2.0", "result":{"1":{"option":"UPDATED", "status":"OK", "severity":"INFO","lastChangeTime":"201503231758", "hostId":"1", "hostName":"exampleName", "brief":"example brief", "extendedInfo": "sample extended info"}},"id":1}
```

### reqTerminate(notification)

HatoholサーバーからHatoholサーバーとHAPの接続を終了することをリクエストします。Hatoholサーバー上から監視サーバーの情報が削除された際に送ることを標準的な動作とします。

***params***

reqTerminateメソッドには引数が存在しません。paramsをnullオブジェクトにして送信してください。

## errorCode

リクエストに成功した場合，送信したリクエストに応じたresultオブジェクトが返されます。
リクエストに失敗した場合，resultオブジェクトではなくerrorオブジェクトが返されます。
このセクションではエラーオブジェクトとして返ってくるエラーコードとエラーメッセージについて解説します。

※各メソッド定義後に埋めていく

|code|message|meaning|
|:--|:-------|:---|
|1  |Unknown error|原因不明のエラーが起こっています|

## 表

### armInfoStatus

|ステータス|解説|
|:---------|:---|
|"INIT"   |初期状態。まだ通信を行っていない|
|"OK"     |通信に成功している|
|"FAILURE"|通信に失敗している|

### serverType

以下が既存のHAPを識別する際に使用されている各サーバータイプのURLとUUIDです。あなたが新しくHAPを作成する場合，サーバータイプを新しく定義する必要があります。以下のUUIDと被っていない任意のUUIDを生成し，Hatoholサーバーにて定義し，使用してください。定義の方法については以下のドキュメントをご覧ください。[[ほげほげ](#http://なんちゃらかんちゃら)]

|名前|UUID|URL|
|:---|:---|:--|
|Zabbix      |8e632c14-d1f7-11e4-8350-d43d7e3146fb||
|Nagios      |902d955c-d1f7-11e4-80f9-d43d7e3146fb||
|Ceilometer  |aa25a332-d1f7-11e4-80b4-d43d7e3146fb||

### triggerSeverity

トリガーの種別です。任意のステータスを各トリガーに設定してください。

|種別|
|:---|
|"ALL"      |
|"UNKNOWN"  |
|"INFO"     |
|"WARNING"  |
|"ERROR"    |
|"CRITICAL" |
|"EMERGENCY"|

### triggerStatus

|ステータス|解説|
|:---------|:---|
|"ALL"    |初期状態。まだ通信を行っていない|
|"OK""    |通信に成功している|
|"PROBLEM"|通信に失敗している|
|"UNKNOWN"|状態不明|

### option

|種類|解説|
|:---------|:---|
|"ALL"    |各データ全てを送信します。Hatoholサーバー内の古いデータを削除し，その後送信した全てのデータを登録します|
|"UPDATED"|アップデートされたデータのみをHatoholサーバーに送信し，同一IDのデータは上書き，初出のデータは新規登録します|

### eventType

イベントのタイプです。任意のタイプを各イベントに設定して下さい。

|タイプ|解説|
|:-----|:--:|
|"GOOD"   |正常|
|"BAD"    |異常|
|"UNKNOWN"|不明|

### itemValueType

アイテムのタイプです。任意のタイプを各アイテムに設定して下さい。

|タイプ|解説|
|:-----|:---|
|"UNKNOWN"|未定義  |
|"FLOAT"  |float型 |
|"INTEGER"|int型   |
|"STRING" |string型|

### result

send~~~コマンドを送信した際におけるDB書き込みの成否です。
書き込みに失敗した際は，再度送信するといった動作が標準的です。

|ステータス|解説|
|:---------|:---|
|"SUCCESS"|DBへの書き込みが成功しています|
|"FAILED"|DBへの書き込みに失敗しています|
