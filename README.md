# HAP(Hatohol Arm Plugin)

## 概要

HAPはJSON-RPC 2.0を用い、Hatoholサーバーと通信を行います。
JSON-RPCの仕様については[公式リファレンス](http://www.jsonrpc.org/specification)をお読みください。

 - 現在、Hatoholはutf-8を標準的な文字コードとし、作成されています。utf-8を使用することを推奨します。
 - リクエスト・レスポンスで使用するIDには、十分なランダム性が必要。
 - HAPの時刻フォーマットはyyyyMMDDHHmmss.nsのstring型。
 - 小数点以下の時刻については省くことが出来ます。また、小数点以下には9桁までしか値を挿入することはできません。小数点以下を省いた場合、または小数点以下が9桁未満の場合には余った桁部に0が挿入されます。(Ex.100 -> 100.000000000, 100.1234 -> 100.123400000)。
 - JSON-RPCにはバッチリクエストという、複数のリクエストを同時に送信する文法も存在しますが、Hatoholはこの文法を用いたリクエストには対応していません。
 - number型の値には正の整数を用いてください。浮動小数点数や負の整数を用いることは出来ません。
 - 通信の方向がHAP -> SRVとなっているコマンド内のnumber型の範囲は0~2147483647。 
 - オブジェクトの名前として各要素のID、値としてそのIDに対応している各値を更にオブジェクト形式で記述してください。ここに記述するIDはユニークである必要があります。

## コマンド一覧

|コマンド名               |解説|HAP -> SRV|SRV -> HAP|メソッドタイプ|
|:------------------------|:---|:--------:|:--------:|:-------------:|
|[getMonitoringServerInfo](#user-content-getMonitoringServerInfo)|監視サーバーの接続情報やポーリング間隔等を取得します|Yes|-|method|
|[getTimestampOfLastTrigger](#user-content-getTimestampOfLastTrigger)|Hatoholサーバーに保存されている最新トリガーのタイムスタンプを取得します|Yes|-|method|
|[getLastEventId](#user-content-getLastEventId)|Hatoholサーバーに保存されている最新イベントのIDを取得します|Yes|-|method|
|[getTimeOfLastEvent](#user-content-getTimeOfLastEvent)|Hatoholサーバーに保存されている最新イベントの発生時間を取得します|Yes|-|method|
|[getIfHostsChanged](#user-content-getIfHostsChanged)|直前のsendHostsによってHatoholサーバー内のホスト情報が変更の真偽を取得します|Yes|-|method|
|[sendUpdatedTriggers](#user-content-sendUpdatedTriggers)|アップデートされたトリガーをHatoholサーバーに送信します|Yes|-|notification|
|[sendHosts](#user-content-sendHosts)|監視サーバーが監視しているホスト一覧をHatoholサーバーに送信します|Yes|-|notification|
|[sendHostGroupElements](#user-content-sendHostGroupElements)|ホストのホストグループ所属情報をHatoholサーバーに送信します|Yes|-|notification|
|[sendHostGroups](#user-content-sendHostGroups)|ホストグループの情報をHatoholサーバーに送信します|Yes|-|notification|
|[sendUpdatedEvents](#user-content-sendUpadtedEvents)|アップデートされたイベントをHatoholサーバーに送信します|Yes|-|notification|
|[sendHapSelfTriggers](#user-content-sendHapSelfTriggers)|HAP自身のトリガーをHatoholサーバーに送信します|Yes|-|notification|
|[sendArmInfo](#user-content-sendArmInfo)|HAPの接続情報をHatoholサーバーに送信します|Yes|-|notification|
|[sendAllTriggers](#user-content-sendAllTrigges)|全てのトリガーをHatoholサーバーに送信します|Yes|-|notification|
|[reqFetchItems](#user-content-reqFetchItems)|HatoholサーバーからHAPへアイテム一覧取得のリクエストを送信します|-|Yes|method|
|[reqTerminate](#user-content-reqTerminate)|HAPとHatoholサーバーとの接続を終了させます|-|Yes|method|
|[reqFetchHistory](#user-content-reqFetchHistory)|HatoholサーバーからHAPへヒストリー一覧取得のリクエストを送信します|-|Yes|method|
|[reqFetchTriggers](#user-content-reqFetchTriggers)|HatoholサーバーからHAPへトリガー一覧取得のリクエストを送信します|-|Yes|method|

## 各コマンド解説

### getMonitoringServerInfo(method)

**params**

getMonitoringServerInfoメソッドには引数が存在しません。nullオブジェクトとしてparamsを送信してください。

**result**

|名前|型|Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:----------:|:---------:|:------:|:---|
|serveId|number|Yes|-|正の整数|監視対象のサーバーID|
|hostName|string|Yes|-|65535byte以内|監視対象のホスト名|
|type|number|Yes|-|0-4|監視サーバーの種類 [[一覧](#user-content-serverType)]|
|ipAddress|string|Yes|-|IPV4, IPV6の取りうる値|監視サーバーのIPアドレス|
|nickName|string|Yes|-|65535byte以内|監視サーバーのニックネーム|
|userName|string|Yes|-|65535byte以内|監視サーバーのユーザーネーム|
|password|string|Yes|-|65535byte以内|監視サーバーのパスワード|
|dbName|string|Yes|-|65535byte以内|監視対象のデータベースのパスワード|
|port|number|Yes|-|0~65535|監視サーバーのポート|
|pollingIntervalSec|number|Yes|-|正の整数|ポーリングを行う間隔|
|retryIntervalSec|number|Yes|-|正の整数|ポーリングが失敗した場合、リトライを行うまでの間隔|

### getTimestampOfLastTrigger(method)

**params**

getTimestampOfLastTriggerメソッドには引数が存在しません。nullオブジェクトとしてparamsを送信してください。

**result**

|名前|型 |Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:-------:|:----------:|:------:|:---|
|timestampOfLastTrigger|string|Yes|-|65535byte以内|最後に更新されたトリガーのタイムスタンプ|

### getLastEventId(method)

**params**

getLastEventIdメソッドには引数が存在しません。nullオブジェクトとしてparamsを送信してください。

**result**

|名前|型 |Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:-------:|:----------:|:------:|:---|
|lastEventId|number|Yes|-|65535byte以内|Hatoholサーバー内に保存されているイベントの中の最新のイベントID|

### getLastTimeOfEvent(method)

同時刻に発生したイベントが存在する場合、取得するイベントが重複する可能性があります。

**params**

getLastTimeOfEventメソッドには引数が存在しません。paramsをnullオブジェクトにして送信してください。

**result**

|名前|型 |Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:-------:|:----------:|:------:|:---|
|lastTimeOfEvent|number|Yes|-|65535byte以内|Hatoholサーバーに保存されている最新イベントの発生時間を取得します|

### getIfHostsChanged(method)

**params**

getIfHostsChangedメソッドには引数が存在しません。paramsをnullオブジェクトにして送信してください。

**result**

|名前|型 |Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:-------:|:----------:|:------:|:---|
|type|boolean|Yes|-|-|監視サーバーが監視しているホストが変更されたかを判断します|

### sendUpdatedTriggers(notification)

**params**

名前：トリガーID

値：

|名前|型 |Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:-------:|:----------:|:------:|:---|
|status        |number|Yes|-|正の整数     |トリガーのステータス [[一覧](#user-content-triggerStatus)]|
|severity      |number|Yes|-|正の整数     |トリガーの種別 [[一覧](#user-content-triggerSeverity)]|
|lastChangeTime|string|Yes|-|65535byte以内|トリガーが最後に更新された時間|
|hostId        |string|Yes|-|正の整数     |トリガーが所属するホストID|
|hostName      |string|Yes|-|65535byte以内|トリガーが所属するサーバーのホスト名|
|brief         |string|Yes|-|65535byte以内|トリガーの概要|
|extendedInfo  |string|Yes|-|65535byte以内|上記の情報以外の必要な情報。主にWebUI上にデータを表示する際に用いられる|

### sendHosts(notification)

**params**

オブジェクトの名前：ホストID

オブジェクトの値：

|名前|型 |Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:-------:|:----------:|:------:|:---|
|hostName|string|Yes|-|65535byte以内|監視サーバーが監視しているホスト名|

### sendHostGroupElements(notification)

**params**

オブジェクトの名前：ホストID

オブジェクトの値：

|名前|型 |Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:-------:|:----------:|:------:|:---|
|groupId|number|Yes|- |正の整数|監視ホストが所属しているグループのID|

### sendHostGroups(notification)

**params**

オブジェクトの名前：グループID

オブジェクトの値：

|名前|型 |Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:-------:|:----------:|:------:|:---|
|groupName|string|Yes|-|65535byte以内|監視ホストが所属するホストグループ|

### sendUpdatedEvents(notification)

**params**

オブジェクトの名前：イベントID

オブジェクトの値：

|名前|型 |Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:-------:|:----------:|:------:|:---|
|time        |string|Yes|-|65535byte以内|イベントが発生した時刻|
|type        |string|Yes|-|65535byte以内|イベントのタイプ [[一覧](#user-content-eventType)]|
|triggerId   |number|Yes|-|正の整数     |このイベントを発火させたトリガーID|
|status      |number|Yes|-|正の整数     |トリガーのステータス [[一覧](#user-content-triggerStatus)]|
|severity    |number|Yes|-|正の整数     |トリガーの種別 [[一覧](#user-content-triggerSeverity)]|
|hostId      |string|Yes|-|65535byte以内|イベントが発生したホストのID|
|hostName    |string|Yes|-|65535byte以内|イベントが発生したホストの名前|
|brief       |string|Yes|-|65535byte以内|イベントの説明。Web上に表示される情報|
|extendedInfo|string|Yes|-|65535byte以内|briefには書いていない追加の情報を記述できます|

### sendHapSelfTriggers(notification)

**params**

名前：サーバーID

値：

|名前         |型|Mandatory|デフォルト値|値の範囲|解説|
|:------------|:----|:----:|:----------:|:------:|:---|
|status        |number|Yes|-|正の整数     |トリガーのステータス[[一覧](#user-content-triggerStatus)]|
|severity      |number|Yes|-|正の整数     |トリガーの種別|
|lastChangeTime|string|Yes|-|65535byte以内|トリガーが最後に更新された時間|
|hostId        |number|Yes|-|正の整数     |監視サーバー内で設定されているホストID|
|hostName      |string|Yes|-|65535byte以内|トリガーが所属するサーバーのホスト名|
|brief         |string|Yes|-|65535byte以内|トリガーの概要|
|extendedInfo  |string|Yes|-|65535byte以内|briefには書いていない追加情報を記述することができます|

### sendArmInfo(notification)

**params**

オブジェクトの名前：サーバーID

オブジェクトの値：

|名前|型|Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:---------:|:----------:|:------:|:---|
|running        |boolean|Yes|-|-|HAPが動作ステータス。<br>trueであれば正常に動作中、falseであるならば異常が発生している|
|status         |string |Yes|-|-|最後のポーリングリザルト [[一覧](#user-content-armInfoStatus)]|
|failureReason  |string |Yes|-|65535byte以内|更新が失敗した理由| 
|lastSuccessTime|string |Yes|-|65535byte以内|最後に更新が成功した時刻|
|lastFailureTime|string |Yes|-|65535byte以内|最後に更新が失敗した時刻|
|numSuccess     |number |Yes|-|正の整数|HAPが起動してから更新に成功した回数| 
|numFailure     |number |Yes|-|正の整数|HAPが起動してから更新に失敗した回数|

```
{"jsonrpc":"2.0", "method":"sendArmInfo", "params":{"1":{"running":true, "status":"INIT", "failureReason":"Example reason", "lastSuccessTime":"201503131611", "lastFailureTime":"201503131615", "numSuccess":165, "numFailure":10}}}
```

### sendAllTrigger(notification)

**params**

名前：サーバーID

値：

|名前         |型|Mandatory|デフォルト値|値の範囲|解説|
|:------------|:----|:----:|:----------:|:------:|:---|
|status        |number|Yes|-|正の整数     |トリガーのステータス [[一覧](user-content-triggerStatus)]|
|severity      |number|Yes|-|正の整数     |トリガーの種別 [[一覧](#user-content-triggerSeverity)]|
|lastChangeTime|string|Yes|-|65535byte以内|トリガーが最後に更新された時間|
|hostId        |number|Yes|-|正の整数     |監視サーバー内設定されているホストID|
|hostName      |string|Yes|-|65535byte以内|トリガーが所属するサーバーのホスト名|
|brief         |string|Yes|-|65535byte以内|トリガーの概要|
|extendedInfo  |string|Yes|-|65535byte以内|briefには書いていない追加情報を記述することができます|

### reqFetchItems(method)

**params**

reqFetchItemsメソッドには引数が存在しません。paramsをnullオブジェクトにして送信してください。

**result**

|名前|型|Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:----------:|:---------:|:------:|:---|
|serverId|number|Yes|-|正の整数|アイテムが所属するサーバーのID|
|hostId|number|Yes|-|正の整数|アイテムが所属するホストのID|
|brief|string|Yes|-|255文字以内|各アイテムの概要|
|lastValueTime|string|Yes|-|255文字以内|各アイテムが最後に更新された時間|
|lastValue|string|Yes|-|255文字以内|各アイテムが最後に更新された際の値|
|prevValue|string|Yes|-|255文字以内|各アイテムが最後に更新される前の値|
|itemGroupName|string|Yes|-|255文字以内|アイテムをグループ分けしたもの<br>任意のグループ名をご使用ください|
|unit|string|Yes|-|255文字以内|valueの単位|

### reqTerminate(notification)

**params**

reqTerminateメソッドには引数が存在しません。paramsをnullオブジェクトにして送信してください。

### reqFetchHistory(method)

**params**

名前:サーバーID

値：

|名前|型 |Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:-------:|:----------:|:------:|:---|
|hostId   |string|Yes|-|255文字以内|ヒストリーのアイテムが所属しているホストID|
|itemId   |number|Yes|-|正の整数   |ヒストリーのアイテムID|
|valueType|string|Yes|-|255文字以内|取得したヒストリーのアイテムタイプ [[一覧](#user-content-itemValueType)]|
|beginTime|string|Yes|-|255文字以内|ヒストリー取得の始点時間を指定します|
|endTime  |string|Yes|-|255文字以内|ヒストリー取得の終点時間を指定します|

**result**

|名前|型 |Mandatory|デフォルト値|値の範囲|解説|
|:---|:--|:-------:|:----------:|:------:|:---|
|itemId|number|Yes|-|正の整数   |ヒストリーのアイテムID|
|value |string|Yes|-|255文字以内|clock時点でのアイテムの値|
|clock |string|Yes|-|255文字以内|このヒストリーでの時刻|

### reqFetchTriggers(method)

**params**

  reqFetchTriggersメソッドには引数が存在しません。paramsをnullオブジェクトにして送信してください。

**result**

|名前         |型|Mandatory|デフォルト値|値の範囲|解説|
|:------------|:----|:----:|:----------:|:------:|:---|
|status        |number|Yes|-|正の整数     |トリガーのステータス|
|severity      |number|Yes|-|正の整数     |トリガーの種別 [[一覧](#user-content-triggerSeverity)]|
|lastChangeTime|string|Yes|-|65535byte以内|トリガーが最後に更新された時間|
|hostId        |number|Yes|-|正の整数     |監視サーバー内で設定されているホストID|
|hostName      |string|Yes|-|65535byte以内|トリガーが所属するサーバーのホスト名|
|brief         |string|Yes|-|65535byte以内|トリガーの概要|
|extendedInfo  |string|Yes|-|65535byte以内|上記の情報以外の必要な情報。主にWebUI上にデータを表示する際に用いられる|

## エラーコード

リクエストに成功した場合は送信したリクエストに対応したresultが、
リクエストに失敗した場合はresultではなくerrorが返ってきます。
このセクションではこのエラーコードを解説します。

|エラーコード|解説|
|:-----------|:---|
|OK                  |エラーが発生せずに無事リクエストが受け入れられ、レスポンスが返ってきたことを表す|
|NOT_FOUND_QUEUE_ADDR|設定したIPアドレス先にAMQP broakerのキューが存在しない|
|UNAVAILABLE_HAP     |HAPプロセスがHatoholサーバーに受け入れられていません|
|UNKNOWN             |原因不明のエラーが起こっています|

## 表

### armInfoStatus

|ステータス|解説|
|:---------|:---|
|INIT   |初期状態。まだ通信を行っていない|
|OK     |通信に成功している|
|FAILURE|通信に失敗している|

### triggerSeverity

トリガーの種別です。任意のステータスを各トリガーに設定してください。

|種別|
|:---|
|ALL      |
|UNKNOWN  |
|INFO     |
|WARNING  |
|ERROR    |
|CRITICAL |
|EMERGENCY|

### triggerStatus

|ステータス|解説|
|:---------|:---|
|ALL    |初期状態。まだ通信を行っていない|
|OK     |通信に成功している|
|PROBLEM|通信に失敗している|
|UNKNOWN|状態不明|

### eventType

イベントのタイプです。任意のタイプを各イベントに設定して下さい。

|タイプ|解説|
|:-----|:--:|
|GOOD   |正常|
|BAD    |異常|
|UNKNOWN|不明|

### itemInfoValueType

イベントのタイプです。任意のタイプを各イベントに設定して下さい。

|タイプ|解説|
|:-----|:---|
|UNKNOWN|未定義  |
|FLOAT  |float型 |
|INTEGER|int型   |
|STRING |string型|

