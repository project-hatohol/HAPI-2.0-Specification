# Hatohol Arm Plugin Interface 2.0 仕様書(2015/04/16)

## ToDO

serverTypeのURL

## 概要

Hatohol Arm Plugin Interface (HAPI) 2.0 は，Hatoholサーバーと監視サーバープラグイン間の情報交換のためのプロトコルです。
両者の間に確立された通信路上で実装されるJSON-RPCのアプリケーションとして構築されます。

以下の図は，上記概要を図で表したものです。

![overview](hapi_overview.png)

## 用語

|用語|説明|
|:---|:---|
|SRV|Hatoholサーバー|
|HAP|Hatohol監視サーバープラグイン|

"MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY" および "OPTIONAL" は，[RFC2119](http://www.ietf.org/rfc/rfc2119.txt)に従います。

## 関連プロトコル

### 通信路

通信路として，Advanced Message Queuing Protocol (AMQP) 0.9.1を用います。
- AMQP 0.9.1: https://www.rabbitmq.com/resources/specs/amqp0-9-1.pdf

### JSON-RPC

情報交換の基本プロトコルとして，JSON-RPC 2.0を用います。
- [JSON-RPC 2.0 (2013-01-04)](http://www.jsonrpc.org/specification)
- [JSON(RFC4627)](https://www.ietf.org/rfc/rfc4627.txt)

#### 注意事項
- 文字エンコードはUTF-8とします(MUST)。NFC正規化すべきです(SHOULD)。文字列はエスケープ表現してもかまいません(MAY)。
- リクエスト・レスポンスで使用するIDオブジェクトの値には，十分なランダム性が必要です(SHOULD)。
- HAPI2.0では，JSON-RPCのバッチリクエストを使用してはなりません(MUST NOT)。

## 動作概要

以下のシーケンス図は，HatoholサーバーとHAP間で送受信される各プロシージャのリクエスト・レスポンスの想定される標準的な動作を表す一例です。

```

Hatoholサーバー                                   HAP
    |                                               
    |                                        Turn on HAP
    |                                              |
    |<----------exchangeProfile(リクエスト)------->|
    |<----------exchangeProfile(レスポンス)------->|
    |                                          |   |
    |                               ポーリング間隔 |
    |                                          |   |
    |<-----getMonitoringServerInfo(リクエスト)-----|
    |------getMonitoringServerInfo(レスポンス)---->|
    |                                              |
    |<-----------getLastInfo(リクエスト)-----------|
    |------------getLastInfo(レスポンス)---------->|
    |<-----------updateHosts(リクエスト)-----------|
    |------------updateHosts(レスポンス)---------->|
    |                                              |
    |<-----------getLastInfo(リクエスト)-----------|
    |------------getLastInfo(レスポンス)---------->|
    |<--------updateHostGroups(リクエスト)---------|
    |---------updateHostGroups(レスポンス)-------->|
    |                                              |
    |<-----------getLastInfo(リクエスト)-----------|
    |------------getLastInfo(レスポンス)---------->|
    |<----updateHostGroupMembership(リクエスト)----|
    |-----updateHostGroupMembership(レスポンス)--->|
    |                                              |
    |<-----------getLastInfo(リクエスト)-----------|
    |------------getLastInfo(レスポンス)---------->|
    |<-----------updateTriggers(リクエスト)--------|
    |------------updateTriggers(レスポンス)------->|
    |                                              |
    |<-----------getLastInfo(リクエスト)-----------|
    |------------getLastInfo(レスポンス)---------->|
    |<-----------updateEvents(リクエスト)----------|
    |------------updateEvents(レスポンス)--------->|
    |                                              |
    |<-----------getLastInfo(リクエスト)-----------|
    |------------getLastInfo(レスポンス)---------->|
    |<-----------updateHostParent(リクエスト)------|
    |------------updateHostParent(レスポンス)----->|
    |                                              |
    |<-----------updateArmInfo(リクエスト)---------|
    |------------updateArmInfo(レスポンス)-------->|
    |                                          |   |
    |                               ポーリング間隔 |
    |                                          |   |
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    |                                              |
    |-----------fetchItems(リクエスト)------------>|
    |<----------fetchItems(レスポンス)-------------|
    |<------------putItems(リクエスト)-------------|
    |-------------putItems(レスポンス)------------>|
    |                                              |
    |-----------fetchHistory(リクエスト)---------->|
    |<----------fetchHistory(レスポンス)-----------|
    |<------------putHistory(リクエスト)-----------|
    |-------------putHistory(レスポンス)---------->|
    |                                              |
    |-----------fetchTriggers(リクエスト)--------->|
    |<----------fetchTriggers(レスポンス)----------|
    |<---------updateTriggers(リクエスト)----------|
    |            "ALL"オプションを使用する         |
    |----------updateTriggers(レスポンス)--------->|
    |                                              |
    |-------------fetchEvents(リクエスト)--------->|
    |<------------fetchEvents(レスポンス)----------|
    |<-----------updateEvents(リクエスト)----------|
    |------------updateEvents(レスポンス)--------->|
    |                                              |

```
## データ型

 - JSON-RPCではなくHatoholが独自に定義しているデータ型について解説します。これらは内部的にはJSON-RPCが定義しているデータ型を使用しています。

|名前|JSON型|解説|
|:---|:---------|:---|
|TimeStamp|string|この型は時刻を格納します。フォーマットはYYYYMMDDhhss.nnnnnnnnnです<br>YYYY,MM,DD,hh,ss,およびnnnnnnnnnは，それぞれ西暦，月，日，時，分，秒，およびナノ秒を表します<br>小数点以下の時刻については省略できます<br>また，小数点以下には9桁までしか値を挿入できません<br>小数点以下を省略した場合，または小数点以下が9桁未満の場合には余った桁部に0が挿入されます<br>(Ex.100 -> 100.000000000, 100.1234 -> 100.123400000)|
|Boolean|true, false|この型は真偽値を格納します。true or falseを指定し，その値の真偽を示します|
|Number|number|number型の値を格納します。この型を指定された要素では，特記しない限り，その値を0~2147483647の範囲に収める必要があります|
|String255|string|string型の値を格納します。この型を指定された要素では，その文字数を255文字以内にしてください|
|String2047|string|string型の値を格納します。この型を指定された要素では，その文字数を2047文字以内にしてください|
|String32767|string|string型の値を格納します。この型を指定された要素では，その文字数を32767文字以内にしてください|

文字数は最終的な表現にかかわらず、NFC正規化後のUTF-32コードポイント数で数えます(MUST)。

## 起動時の動作について

 - HAPやHatoholサーバーの起動，または再起動直後に自身と接続相手が使用可能なプロシージャの一覧をexchangeProfileプロシージャを用いて交換します。この情報を基に，互いが使用するプロシージャの最適化を行うことができます。

## プロシージャ

 - 「M/O」はそのプロシージャがMandatory(必須)かOptional(任意)であるかを表します。Mandatoryであるプロシージャは実装を省略できません。

### Hatoholサーバーに実装するプロシージャ

|プロシージャ名|解説|タイプ|M/O|
|:-------------|:---|:-----|:-:|
|[exchangeProfile](#user-content-exchangeprofilemethod)|HAPが実装しているプロシージャ一覧とHAPの名前を受け取り，そのレスポンスとして自身が実装しているプロシージャ一覧と自身の名前を返します|method|M|
|[getMonitoringServerInfo](#user-content-getmonitoringserverinfomethod)|HAPとの接続情報やポーリング間隔等をHAPに返します|method|M|
|[getLastInfo](#user-content-getlastinfomethod)|リクエストで指定された要素の最新情報をHAPに返します|method|M|
|[putItems](#user-content-putitemsmethod)|HAPが監視しているアイテム一覧を受け取ります|method|O|
|[putHistory](#user-content-puthistorymethod)|HAPが監視している各アイテムのヒストリーを受け取ります|method|O|
|[updateHosts](#user-content-updatehostsmethod)|HAPからホスト一覧を受け取り，更新します|method|O|
|[updateHostGroups](#user-content-updatehostgroupsmethod)|HAPからホストグループ一覧を受け取り，更新します|method|O|
|[updateHostGroupMembership](#user-content-updatehostgroupmembershipmethod)|HAPからホストグループ所属情報を受け取り，更新します|method|O|
|[updateTriggers](#user-content-updatetriggersmethod)|HAPからトリガー情報を受け取り，更新します|method|O|
|[updateEvents](#user-content-updateeventsmethod)|HAPからイベント情報を受け取り，更新します|method|O|
|[updateHostParent](#user-content-updatehostparentmethod)|HAPが監視しているホスト同士のVM親子関係を更新します|method|O|
|[updateArmInfo](#user-content-updatearminfomethod)|HAPの接続ステータスを更新します|method|M|

### HAPに実装するプロシージャ

|プロシージャ名|解説|タイプ|M/O|
|:-------------|:---|:-----|:-:|
|[exchangeProfile](#user-content-exchangeprofilemethod)|Hatoholサーバーが実装しているプロシージャ一覧とHatoholサーバーの名前を受け取り，そのレスポンスとして自身が実装しているプロシージャ一覧と自身の名前を返します|method|O|
|[fetchItems](#user-content-fetchitemsmethod)|Hatoholサーバーからのアイテム取得リクエストを受け入れます|method|O|
|[fetchHistory](#user-content-fetchhistorymethod)|Hatoholサーバーからのヒストリー取得リクエストを受け入れます|method|O|
|[fetchTriggers](#user-content-fetchtriggersmethod)|Hatoholサーバーからのトリガー取得リクエストを受け入れます|method|O|
|[fetchEvents](#user-content-fetcheventsmethod)|Hatoholサーバーからのイベント取得リクエストを受け入れます|method|O|

### exchangeProfile(method)

 - Hatoholサーバー，HAP両者に共通するプロシージャです。主に初期起動時，再起動時に使用することを標準的な動作とします。詳細については[起動時の動作について](#user-content-起動時の動作について)をご覧ください。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|procedures|String255配列|M|-|送信先が使用可能なプロシージャ一覧|
|name      |String255    |M|-|送信先のプロセス名です。接続完了の旨を伝えるログなどに利用されます|

```
{"jsonrpc":"2.0", "method":"exchangeProfile", "params":{"procedures":["getMonitoringServerInfo", "getLastInfo", "putItems", "updateArmInfo", "fetchItems"], "name":"exampleName"} "id":1}
```

***リザルト(result)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|procedures|String255配列|M|-|送信先が使用可能なプロシージャ一覧|
|name      |String255    |M|-|送信先のプロセス名です。接続完了の旨を伝えるログなどに利用されます|

```
{"jsonrpc":"2.0", "result":{"procedures":["getMonitoringServerInfo", "getLastInfo", "putItems", "updateArmInfo", "fetchItems"],"name":"exampleName"} "id":1}
```

### getMonitoringServerInfo(method)

 - ポーリング時間毎にHatoholサーバーに自身の接続情報やポーリング間隔等を問い合わせることを標準的な動作としますが，任意のタイミングで問い合わせることもできます。

***リクエスト(params)***

 - getMonitoringServerInfoメソッドには引数が存在しません。paramsオブジェクトの値を空文字にしてリクエストを送信してください。

```
{"jsonrpc":"2.0", "method":"getMonitoringServerInfo", "params":"", "id":1}
```

***リザルト(result)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|serverId          |Number     |M|-|監視サーバーのサーバーID|
|url               |String2047 |M|-|監視サーバーのURL [[解説](#user-content-servertype)]|
|type              |string     |M|-|監視サーバーの種類 [[一覧](#user-content-servertype)]|
|nickName          |String255  |M|-|監視サーバーのニックネーム|
|userName          |String255  |M|-|監視サーバーのユーザーネーム|
|password          |String255  |M|-|監視サーバーのパスワード|
|pollingIntervalSec|Number     |M|-|ポーリングを行う間隔|
|retryIntervalSec  |Number     |M|-|ポーリングが失敗した場合，リトライを行うまでの間隔|
|extendedInfo      |String32767|M|-|プラグイン固有の情報を格納することができる|

```
{"jsonrpc":"2.0", "result":{"serverId":1, "url":"http://example.com:80", "type":0, "nickName":"exampleName", "userName":"Admin", "password":"examplePass", "pollingIntervalSec":30, "retryIntervalSec":10, "extendedInfo":"exampleExtraInfo"}, "id":1}
```

### getLastInfo(method)

 - プロシージャ名にupdateが付いているプロシージャの呼び出し時にHatoholサーバーに送信，保存されたlastInfo情報を要求します。取得したlastInfoを用いて，前回までに送信したデータと現在所持しているデータの差分をHatoholサーバーに送信できます。
 - 初回起動時など，HatoholサーバーにlastInfoが保存されていない場合，resultオブジェクトの値は空文字として返ってきます。

***リクエスト(params)***

|paramsオブジェクトの値|型 |M/O|デフォルト値|解説|
|:---------------------|:--|:-:|:----------:|:---|
|指定要素|String255|M|-|どの種類のlastInfoが必要かを指定する必要があります。以下の表を参照してください|

|paramsオブジェクトの値一覧|解説|
|:-------------------------|:---|
|"host"               |ホストの最新情報を指定します。|
|"hostGroup"          |ホストグループの最新情報を指定します。|
|"hostGroupMembership"|ホストグループの所属情報の最新情報を指定します。|
|"trigger"            |トリガーの最新情報を指定します。|
|"event"              |イベントの最新情報を指定します。|
|"hostParent"         |ホストのVM親子関係の最新情報を指定します。|

```
{"jsonrpc":"2.0", "method":"getLastInfo", "params":"trigger", "id":1}
```

***リザルト(result)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|最新情報|String255|M|-|Hatoholサーバーに保存されている指定した要素の最新情報|

```
{"jsonrpc":"2.0", "result":"201504011349", "id":1}

この例ではlastInfoとしてタイムスタンプが返ってきています
```

### putItems(method)

 - Hatoholサーバーとの接続完了時，または[fetchItems](#user-content-fetchitemsmethod)プロシージャのリクエストをHatoholサーバーから受け取った時に全てのアイテム情報をHatoholサーバーへ送信することを標準動作とします。Hatoholサーバーの負荷が高くなることが危惧されるため，任意のタイミングで使用することはできません。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|items  |object   |M|-|アイテム情報を格納するオブジェクトを配置します。詳細は次のテーブルを確認してください|
|fetchId|String255|M|-|Hatoholサーバーから送られたどのfetchItemsプロシージャによるリクエストに対するレスポンスであるかを示すIDです。fetchItemsプロシージャのparams内のfetchIdオブジェクトの値をここに入れてください。|

***itemsオブジェクト***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|itemId       |String255    |M|-|アイテムのID|
|hostId       |String255    |M|-|アイテムが所属するホストのID|
|brief        |String255    |M|-|アイテムの概要|
|lastValueTime|TimeStamp    |M|-|アイテムが最後に更新された時刻|
|lastValue    |String255    |M|-|アイテムが最後に更新された際の値|
|itemGroupName|String255配列|M|-|アイテムが所属しているグループ名|
|unit         |String255    |M|-|valueの単位|

```
{"jsonrpc":"2.0","method":"putItems", "params":{"items":[{"itemId":"1", "hostId":"1", "brief":"example brief", "lastValueTime":"201504101755", "lastValue":"example value", "itemGroupName":"example name", "unit":"example unit"}, {"itemId":"2", "hostId":"1", "brief":"example brief", "lastValueTime":"201504101755", "lastValue":"example value", "itemGroupName":"example name", "unit":"example unit"}], "fetchId":"1"}, "id":1}
```

***リザルト(result)***

 - putItemsに返す値はありません。resultオブジェクトの値は空文字としてレスポンスが返ってきます。

```
{"jsonrpc":"2.0", "result":"", "id":1}
```

### putHistory(method)

 - [fetchHistory](#user-content-fetchhistorymethod)プロシージャをHatoholサーバーから受け取った際に，条件にマッチするヒストリーをHatoholサーバーに送信します。Hatoholサーバーの負荷が高くなることが危惧されるため，任意のタイミングで使用することはできません。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|アイテムID|object配列|M|-|ヒストリー情報を格納するオブジェクトを配置します。詳細は次のテーブルを確認してください|
|fetchId   |String255 |M|-|Hatoholサーバーから送られたどのリクエストに対するレスポンスであるかを示すIDです。fetchHistoryのparams内のfetchIdオブジェクトの値をここに入れてください|

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|value |String255|M|-|time時点でのアイテムの値|
|time  |TimeStamp|M|-|このヒストリーの値が記録された時刻|

```
{"jsonrpc":"2.0", "method":"putHistory", "params":{"itemId":"1", "histories": [{"value":"exampleValue","time":"201503231130"},{"value":"exampleValue2","time":"201503231130"}], "fetchId":"1"}, "id":1}
```
***リザルト(result)***

 - putHistoryに返す値はありません。resultオブジェクトの値は空文字としてレスポンスが返ってきます。

```
{"jsonrpc":"2.0", "result":"", "id":1}
```

### updateHosts(method)

 - Hatoholサーバーとの接続完了時，またはHAPが内部的に保存している登録ホスト情報が変更された際は,"ALL"オプションを用いて全てのホスト情報をHatoholサーバーに送信します。
 - "UPDATE"オプションを用いた場合は[getLastInfo](#user-content-getlastinfomethod)プロシージャ，またはHAP自身から呼び出したlastInfoを基に，その時点から現時点までに追加されたホストをHatoholサーバーに送信します。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|hosts       |object配列|M|-|ホスト情報を格納するオブジェクトを配置します。詳細は次のテーブルを確認してください|
|updateType|string    |M|-|送信オプション[[一覧](#user-content-updatetype)]の中から状況に応じた送信オプションを選択してください|
|lastInfo    |String32767 |O|-|最後に送信したホストの情報を送信する。この情報が[getLastInfo](#user-content-getlastinfomethod)の返り値になる|

***hostsオブジェクト***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|hostId  |String255|M|-|監視サーバーが監視しているホストID|
|hostName|String255|M|-|監視サーバーが監視しているホスト名|

```
{"jsonrpc":"2.0","method":"updateHosts", "params":{"hosts":[{"hostId":"1", "hostName":"exampleHostName1"}], "updateType":"UPDATE","lastInfo":"201504091052"}, "id":1}
```

***リザルト(result)***

 - 送信したデータが正常に更新されたかどうかをresultオブジェクトで受け取ります。受け取る値については[[一覧](#user-content-updateresult)]をご覧ください。

```
{"jsonrpc":"2.0", "result":"SUCCESS", "id":1}
```

### updateHostGroups(method)

 - Hatoholサーバーとの接続完了時，またはHAPが内部的に保存している登録ホスト情報が変更された際は"ALL"オプションを用い，全てのホストグループ情報をHatoholサーバーに送信します。
 - "UPDATE"オプションを用いた場合は[getLastInfo](#user-content-getlastinfomethod)プロシージャ，またはHAP自身から呼び出したlastInfoを基に，その時点から現時点までに追加されたホストグループをHatoholサーバーに送信します。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|hostGroups  |object配列|M|-|ホストグループ情報を格納するオブジェクトを配置します。詳細は次のテーブルを確認してください|
|updateType|string    |M|-|送信オプション[[一覧](#user-content-updatetype)]の中から状況に応じた送信オプションを選択してください|
|lastInfo    |String32767|O|-|最後に送信したホストグループの情報を送信する。この情報が[getLastInfo](#user-content-getlastinfomethod)の返り値になる|

***hostGroupsオブジェクト***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|groupId  |String255|M|-|ホストグループのID|
|groupName|String255|M|-|グループIDに対応したホストグループの名前|

```
{"jsonrpc":"2.0","method":"updateHostGroups", "params":{"hostGroups":[{"groupId":"1", "groupName":"Group2"}],"updateType":"ALL", "lastInfo":"201504091049"}, "id":1}
```

***リザルト(result)***

 - 送信したデータが正常に更新されたかどうかをresultオブジェクトで受け取ります。受け取る値については[[一覧](#user-content-updateresult)]をご覧ください。

```
{"jsonrpc":"2.0", "result":"SUCCESS", "id":1}
```

### updateHostGroupMembership(method)

 - Hatoholサーバーとの接続完了時，またはHAPが内部的に保存している登録ホスト情報が変更された際は"ALL"オプションを用い，全てのホストグループ所属情報をHatoholサーバーに送信します。
 - "UPDATE"オプションを用いた場合は[getLastInfo](#user-content-getlastinfomethod)プロシージャ，またはHAP自身から呼び出したlastInfoを基に，その時点から現時点までに追加されたホストグループ所属情報をHatoholサーバーに送信します。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|hostGroupMembership|object配列|M|-|ホストグループ所属情報を格納するオブジェクトを配置します。詳細は次のテーブルを確認してください|
|updateType       |string    |M|-|送信オプション[[一覧](#user-content-updatetype)]の中から状況に応じた送信オプションを選択してください|
|lastInfo           |String32767|O|-|最後に送信したホストグループ所属情報の情報を送信する。この情報が[getLastInfo](#user-content-getlastinfomethod)の返り値になる|

***hostGroupMembershipオブジェクト***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|hostId  |String255    |M|-|ホストのID|
|groupIds|String255配列|M|-|ホストグループのID|
```
{"jsonrpc":"2.0","method":"updateHostGroupMembership", "params":{"hostGroupsMembership":[{"hostId":"1", "groupIds":["1", "2", "5"]}], "lastInfo":"201504091056", "updateType":"ALL"}, "id":1}
```

***リザルト(result)***

 - 送信したデータが正常に更新されたかどうかをresultオブジェクトで受け取ります。受け取る値については[[一覧](#user-content-updateresult)]をご覧ください。

```
{"jsonrpc":"2.0", "result":"SUCCESS", "id":1}
```

### updateTriggers(method)

[getLastInfo](#user-content-getlastinfomethod)を用いて取得，またはHAP自身が保管している最新トリガー情報を基に，そのトリガーから現時点までに更新されたトリガーをHatoholサーバーに送信するか，全てのトリガーを送信します。
 - Hatoholサーバーとの接続完了時，fetchTriggersプロシージャによる要求があった際は"ALL"オプションを用い，全てのトリガーをHatoholサーバーに送信します。
 - "UPDATE"オプションを用いた場合は[getLastInfo](#user-content-getlastinfomethod)プロシージャ，またはHAP自身から呼び出したlastInfoを基に，その時点から現時点までに更新，追加されたトリガーをHatoholサーバーに送信します。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|triggers    |object配列|M|-|トリガー情報を格納するオブジェクトを配置します。詳細は次のテーブルを確認してください|
|updateType|String255 |M|-|送信オプション[[一覧](#user-content-updatetype)]の中から状況に応じた送信オプションを選択してください|
|lastInfo    |String32767|O|-|最新トリガーの情報を送信する。この情報が[getLastInfo](#user-content-getlastinfomethod)の返り値になる|
|fetchId     |String255 |O|-|Hatoholサーバーから送られたどのリクエストに対するレスポンスであるかを示すIDです。fetchTriggersのparams内のfetchIdオブジェクトの値をここに入れてください|

***triggersオブジェクト***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|triggerId     |String255  |M|-|トリガーのID。HAP自身のトリガーを送信する場合は，トリガーIDとホストIDを"_SELF_"と記述することで送信したトリガーをSELFトリガー扱いにできます|
|status        |string     |M|-|トリガーのステータス [[一覧](#user-content-triggerstatus)]|
|severity      |string     |M|-|トリガーの種別 [[一覧](#user-content-triggerseverity)]|
|lastChangeTime|String255  |M|-|トリガーが最後に更新された時刻|
|hostId        |String255  |M|-|トリガーが所属するホストID|
|hostName      |String255  |M|-|トリガーが所属するサーバーのホスト名|
|brief         |String255  |M|-|トリガーの概要|
|extendedInfo  |String32767|M|-|上記の情報以外の必要な情報。主にWebUI上にデータを表示する際に用いられる|

```
{"jsonrpc":"2.0", "method":"updateTriggers", "params":{"updateType":"UPDATED", "lastInfo":"201504061606", "fetchId":"1", "triggers":[{"triggerId":"1", "status":"OK", "severity":"INFO","lastChangeTime":"201503231758", "hostId":"1", "hostName":"exampleName", "brief":"example brief", "extendedInfo": "sample extended info"}]},"id":1}
```

***リザルト(result)***

```
{"jsonrpc":"2.0", "result":"SUCCESS", "id":1}
```

### updateEvents(method)

 - 一度に送信できるイベント数は1000件までです。1000件を越える場合は，複数回に分けて送信してください。
 - イベントIDが重複したイベントを送信することは認められてします。しかし，元から存在するイベントと送信するイベントのどちらが優先されるかは未定義です。
 - 自発的にイベントを送信する動作とfetchEventsプロシージャに対するレスポンスとしてイベントを送信する動作の2つの動作が存在します。
 - 自発的にイベントを送信する場合は，まず[getLastInfo](#user-content-getlastinfomethod)のレスポンスや，HAP自身からlastInfoを取得します。取得したlastInfoで判別したイベントから現時点までに発生した差分のイベントをHatoholサーバーに送信します。初回通信時は，lastInfoの値が空文字であるため，接続以前に発生したイベントをすべて送信するか，何も送信しない選択がHAP作成者に委ねられています。
 - fetchEventsプロシージャのlastInfoで指定されたイベントより先にイベントが存在しない場合は，eventsオブジェクトの値を空配列にして送信してください。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|events     |object配列|M|-|イベント情報を格納するオブジェクトを配置します。詳細は次のテーブルを確認してください。|
|lastInfo   |String32767|O|-|イベントを送信する際，次回イベントを送信する際の基準となる情報を送信する。この情報が[getLastInfo](#user-content-getlastinfomethod)の返り値になる。しかし，mayMoreFlagの値がtrueとなっている場合，この値はHatoholのDBへは保存されずHatoholサーバープロセスに一時的に保存される|
|mayMoreFlag|Boolean   |O|-|fetchEventsプロシージャで指定された件数に満たない件数のイベントを送信し，送信すべきイベントがまだ残っている可能性がある場合に値をtrueとしてください。この値をtrueにする場合，最低限イベントを1件は送信する必要があります|
|fetchId    |String255 |O|-|このオブジェクトはfetchEventsによるリクエストを受けた場合のみ記述する必要があります。Hatoholサーバーから送られたどのリクエストに対するレスポンスであるかを示すIDです。fetchEventsのparams内のfetchIdオブジェクトの値をここに入れてください|

***eventsオブジェクト***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|eventId     |String255  |M|-|イベントのID|
|time        |TimeStamp  |M|-|イベントが発生した時刻|
|type        |string     |M|-|イベントのタイプ [[一覧](#user-content-eventtype)]|
|triggerId   |String255  |O|-|このイベントを発火させたトリガーID。トリガーとイベントを関連付けないことも可能なため，必須ではありません|
|status      |string     |M|-|トリガーのステータス [[一覧](#user-content-triggerstatus)]|
|severity    |string     |M|-|トリガーの種別 [[一覧](#user-content-triggerseverity)]|
|hostId      |String255  |M|-|イベントが発生したホストのID|
|hostName    |String255  |M|-|イベントが発生したホストの名前|
|brief       |String255  |M|-|イベントの説明。Web上に表示される情報|
|extendedInfo|String32767|M|-|briefには書いていない追加の情報を記述できます|

```
{"jsonrpc":"2.0", "method":"updateEvents", "params":{"events":[{"eventId":"1", "time":"201503231513", "type":"GOOD", "triggerId":2, "status": "OK","severity":"INFO":, "hostId":3, "hostName":"exampleName", "brief":"example brief", "extendedInfo": "sampel extended info"}], "lastInfo":"201504011759", "fetchId":"1"},"id":1}
```

***リザルト(result)***

 - 送信したデータが正常に更新されたかどうかをresultオブジェクトで受け取ります。受け取る値については[[一覧](#user-content-updateresult)]をご覧ください。

```
{"jsonrpc":"2.0", "result":"SUCCESS", "id":1}
```

### updateHostParent(method)

 - Hatoholサーバーとの接続完了時は"ALL"オプションを用い，全てのVM親子関係をHatoholサーバーに送信します。
 - "UPDATE"オプションを用いた場合は[getLastInfo](#user-content-getlastinfomethod)プロシージャ，またはHAP自身から呼び出したlastInfoを基に，その時点から現時点までに追加されたVM親子関係をHatoholサーバーに送信します。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|hostParent  |object配列|M|-|VMの親子関係を格納するオブジェクトを配置します。詳細は次のテーブルを確認してください|
|updateType|String255 |M|-|送信オプション[[一覧](#user-content-updatetype)]の中から状況に応じた送信オプションを選択してください|
|lastInfo    |String32767|O|-|最後に送信したホストグループ所属情報の情報を送信する。この情報が[getLastInfo](#user-content-getlastinfomethod)の返り値になる|

***hostParentオブジェクト***

 - VMの親子関係を削除する場合は親ホストIDの値を空文字にすることで，送信した子ホストIDの親子関係をHatoholサーバーから削除することができます。

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|childHostId |String255|M|-|VMの子ホストのID|
|parentHostId|String255|M|-|VMの親ホストのID|

```
{"jsonrpc":"2.0", "method":"updateHostParent", "params":{[{"childHostId":"12","parentHostId":"10"}], "updateType":"ALL", "lastInfo":"201504152246"} "id":1}
```

***リザルト(result)***

```
{"jsonrpc":"2.0", "result":"SUCCESS", "id":1}
```

### updateArmInfo(method)

HostやTrigger，Event情報の送信処理が行われるたびにHatoholサーバーに送信することを標準的な動作としますが，任意に送信してもかまいません。最小間隔は１秒（MUST），最大間隔は[getMonitoringServerInfo](#user-content-getmonitoringserverinfomethod)で取得したポーリング時間の2倍（SHOULD）とします。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|lastStatus         |string   |M|-|最新のポーリング結果 [[一覧](#user-content-arminfostatus)]|
|failureReason      |String255|M|-|情報取得が失敗した理由|
|lastSuccessTime    |TimeStamp|M|-|最後に情報取得が成功した時刻|
|lastFailureTime    |TimeStamp|M|-|最後に情報取得が失敗した時刻|
|numSuccess         |Number   |M|-|HAPが起動してから情報取得に成功した回数|
|numFailure         |Number   |M|-|HAPが起動してから情報取得に失敗した回数|

```
{"jsonrpc":"2.0", "method":"updateArmInfo", "params":{"lastStatus":"INIT", "failureReason":"Example reason", "lastSuccessTime":"201503131611", "lastFailureTime":"201503131615", "numSuccess":165, "numFailure":10}, "id":1}
```

***リザルト(result)***

 - 送信したデータが正常に更新されたかどうかをresultオブジェクトで受け取ります。受け取る値については[[一覧](#user-content-updateresult)]をご覧ください。

```
{"jsonrpc":"2.0", "result":"SUCCESS", "id":1}
```

### fetchItems(method)

Hatoholサーバーがアイテム情報を要求しているときにHAPに送信されます。このプロシージャを受け取った時，resultとしてリクエスト受け入れの成否を返す必要があります。その後，指定されたホストに属する全てのアイテムをputItemsプロシージャ[putItems](#user-content-putitemsmethod)を用いてHatoholサーバーに送信してください。その際，paramsのfetchIdオブジェクトの値を[putItems](#user-contents-putitemsmethod)に渡す必要があります。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|hostIds|String255配列|M|-|ホストを指定し取得するアイテムを限定します|
|fetchId|String255    |M|-|putItemsプロシージャで使用します。そのputItemsプロシージャがどのfetchItemsプロシージャによる要求に対応したものかをHatoholサーバーが識別するために必要です|

```
{"jsonrpc":"2.0", "method":"fetchItems", "params":{"hostIds":["1", "2", "3"], "fetchId":"1"}, "id":1}
```

***リザルト(result)***

 - リクエストの受け入れ成否をresultオブジェクトとしてHatoholサーバーに返す必要があります。返す値については[[一覧](#user-content-fetchresult)]をご覧ください。

```
{"jsonrpc":"2.0", "result":"SUCCESS", "id":1}
```

### fetchHistory(method)

 - このプロシージャは，Hatoholサーバーがヒストリーを要求しているときにHAPに送信されます。HAPはレスポンスとしてリクエスト受け入れの成否を返す必要があります。その後，指定条件に当てはまるヒストリーを[putHistoryプロシージャ](#user-content-puthistorymethod)を用いてHatoholサーバーに送信してください。その際，fetchHistoryプロシージャのparams内にあるfetchIdの値をputHistoryプロシージャに渡す必要があります。
 - paramsオブジェクト内にあるbeginTime，endTimeはbeginTime以上，endTime以下の条件に当てはまるHistory取得することを想定しています。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|hostId    |String255|M|-|ヒストリーのアイテムが所属しているホストID|
|itemId    |String255|M|-|ヒストリーのアイテムID|
|beginTime |TimeStamp|M|-|ヒストリー取得域の始点時刻を指定します|
|endTime   |TimeStamp|M|-|ヒストリー取得域の終点時刻を指定します|
|fetchId   |String255|M|-|putHistoryプロシージャで使用します。そのputHistoryプロシージャがどのfetchHistoryプロシージャによる要求に対応したものかをHatoholサーバーが識別するために必要です|

```
{"jsonrpc":"2.0", "method":"fetchHistory", "params":{"hostId":"1", "itemId":1, "valueType":"INTERGER", "beginTime":"201503231513", "beginTime":"201503231513", "fetchId":1 },"id":1}
```

***リザルト(result)***

 - リクエストの受け入れ成否をresultオブジェクトとしてHatoholサーバーに返す必要があります。返す値については[[一覧](#user-content-fetchresult)]をご覧ください。

```
{"jsonrpc":"2.0", "result":"SUCCESS", "id":1}
```

### fetchTriggers(method)

 - このプロシージャは，Hatoholサーバーがトリガー情報を要求しているときにHAPに送信されます。HAPはレスポンスとしてリクエスト受け入れの成否を返す必要があります。その後，[updateTriggersプロシージャ](#user-content-updatetriggersmethod)の"ALL"オプションを用いて指定されたホストに属する全てのトリガーを送信してください。その際，fetchIdとhostIdsの値をupdateTriggersプロシージャに渡す必要があります。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|hostIds|String255配列|M|-|ホストを指定し取得するトリガーを限定します|
|fetchId|String255    |M|-|updateTriggersプロシージャで使用します。そのupdateTriggersプロシージャがどのfetchTriggersプロシージャによる要求に対応したものかをHatoholサーバーが識別するために必要です|

```
{"jsonrpc":"2.0", "method":"fetchTriggers", "params":{"fetchId":"1"}, "id":1}
```

***リザルト(result)***

 - リクエストの受け入れ成否をresultオブジェクトとしてHatoholサーバーに返す必要があります。返す値については[[一覧](#user-content-fetchresult)]をご覧ください。

```
{"jsonrpc":"2.0", "result":"SUCCESS", "id":1}
```

### fetchEvents(method)

 - 指定したイベントIDから昇順，または降順で指定した件数のイベントを取得するリクエストが送信されます。HAPはレスポンスとしてリクエスト受け入れの成否を返す必要があります。その後，指定された条件のイベントを[updateEventsプロシージャ](#user-content-updateeventsmethod)を用いて送信してください。その際，fetchEventsプロシージャのparamsオブジェクト内にあるfetchIdの値をupdateEventsプロシージャに渡す必要があります。
 - 指定できる取得件数は一度のリクエストで1000件までです。それ以上の件数を取得したい場合は複数回のリクエストに分けて取得してください。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|lastInfo |String32767|M|-|基準となるイベントの情報です|
|count    |Number   |M|-|取得するイベント件数。1000件までしか指定できません|
|direction|String255|M|-|"ASC"（指定したIDより新しいイベント）または”DESC”(指定したIDより古いイベント)を選択します|
|fetchId  |String255|M|-|updateEventsプロシージャで使用します。そのupdateEventsプロシージャがどのfetchEventsプロシージャによる要求に対応したものかをHatoholサーバーが識別するために必要です|

```
{"jsonrpc":"2.0", "method":"fetchEvents", "params":{"fetchId":"1", "lastInfo":"10", "count": "1000", "direction":"ASC"}, "id":1}
```

***リザルト(result)***

 - リクエストの受け入れ成否をresultオブジェクトとしてHatoholサーバーに返す必要があります。返す値については[[一覧](#user-content-fetchresult)]をご覧ください。

```
{"jsonrpc":"2.0", "result":"SUCCESS", "id":1}
```

## 表

### armInfoStatus

|ステータス|解説|
|:---------|:---|
|"INIT"   |初期状態。まだ通信を行っていない|
|"OK"     |通信に成功している|
|"NG"|通信に失敗している|

### ServerType

 - 既存のHAPを識別する際に使用されている各サーバータイプのURLとUUIDです。あなたがHAPを作成する場合，サーバータイプを新しく定義する必要があります。

|名前|UUID|URL|
|:---|:---|:--|
|Zabbix      |8e632c14-d1f7-11e4-8350-d43d7e3146fb||
|Nagios      |902d955c-d1f7-11e4-80f9-d43d7e3146fb||
|Ceilometer  |aa25a332-d1f7-11e4-80b4-d43d7e3146fb||

### triggerSeverity

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
|"OK""    |通信に成功している|
|"NG"|通信に失敗している|
|"UNKNOWN"|状態不明|

### updateType

|種類|解説|
|:---------|:---|
|"ALL"    |各データ全てを送信します。Hatoholサーバー内の古いデータを削除し，その後送信した全てのデータを登録します|
|"UPDATED"|アップデートされたデータのみをHatoholサーバーに送信し，同一IDのデータは上書き，初出のデータは新規登録します|

### eventType

|タイプ|解説|
|:-----|:--:|
|"GOOD"        |正常|
|"BAD"         |異常|
|"UNKNOWN"     |不明|
|"NOTIFICATION"|通知|

### updateResult

 - updateプロシージャをコールした際の更新の成否です。
 - 更新に失敗した際は，再度updateプロシージャをコールするといった動作が標準的です。


|ステータス|解説|
|:---------|:---|
|"SUCCESS"|更新が正常に終了しました|
|"FAILURE" |更新が失敗しました|

### fetchResult

 - fetchと名のついたプロシージャが呼び出された際に，そのリクエストを受け入れたかどうかの成否です。リクエストの受け入れに失敗した際は，再度リクエストするといった動作が標準的です。

|ステータス|解説|
|:---------|:---|
|"SUCCESS"|リクエストの受け入れに成功しました|
|"ABBREV"|リクエストの間隔が近いため，リクエストの受け入れを省略しました|
|"FAILURE"|リクエストの受け入れに失敗しました|

<!--
## 改版履歴
-->

## 連絡先
不明点についてはHatoholコミュニティにお問い合わせください。[hatohol-users@sourceforge.net]

## 著作権
Copyright (C)2015 Project Hatohol
