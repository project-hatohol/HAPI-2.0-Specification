# Hatohol Arm Plugin Interface 2.1 仕様書(2016/03/25)

## 概要

Hatohol Arm Plugin Interface 2.1(以下HAPI2と記す) は，Hatoholサーバーと監視サーバープラグイン間の情報交換のためのプロトコルです。
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
- HAPI2では，JSON-RPCのバッチリクエストを使用してはなりません(MUST NOT)。
- HAPI2では，AMQPのキュー名を動的に生成し使用することを想定していません。Hatoholサーバーと通信を行う際に使用するキューの名前は，予めユーザーが決めておきHatoholサーバー側，プラグイン側の両者がそのキュー名を使用することを想定しています。

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
    |<------------putHosts(リクエスト)-------------|
    |-------------putHosts(レスポンス)------------>|
    |                                              |
    |<-----------getLastInfo(リクエスト)-----------|
    |------------getLastInfo(レスポンス)---------->|
    |<---------putHostGroups(リクエスト)-----------|
    |----------putHostGroups(レスポンス)---------->|
    |                                              |
    |<-----------getLastInfo(リクエスト)-----------|
    |------------getLastInfo(レスポンス)---------->|
    |<-----putHostGroupMembership(リクエスト)------|
    |------putHostGroupMembership(レスポンス)----->|
    |                                              |
    |<-----------getLastInfo(リクエスト)-----------|
    |------------getLastInfo(レスポンス)---------->|
    |<------------putTriggers(リクエスト)----------|
    |-------------putTriggers(レスポンス)--------->|
    |                                              |
    |<-----------getLastInfo(リクエスト)-----------|
    |------------getLastInfo(レスポンス)---------->|
    |<------------putEvents(リクエスト)------------|
    |-------------putEvents(レスポンス)----------->|
    |                                              |
    |<-----------getLastInfo(リクエスト)-----------|
    |------------getLastInfo(レスポンス)---------->|
    |<------------putHostParents(リクエスト)--------|
    |-------------putHostParents(レスポンス)------->|
    |                                              |
    |<------------putArmInfo(リクエスト)-----------|
    |-------------putArmInfo(レスポンス)---------->|
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
    |<----------putTriggers(リクエスト)------------|
    |            "ALL"オプションを使用する         |
    |-----------putTriggers(レスポンス)----------->|
    |                                              |
    |-------------fetchEvents(リクエスト)--------->|
    |<------------fetchEvents(レスポンス)----------|
    |<------------putEvents(リクエスト)------------|
    |-------------putEvents(レスポンス)----------->|
    |                                              |
    |------updateMonitoringServerInfo(通知)------->|
    |                                              |

```
## データ型

 - JSON-RPCではなくHatoholが独自に定義しているデータ型について解説します。これらは内部的にはJSON-RPCが定義しているデータ型を使用しています。

|名前|JSON型|解説|
|:---|:---------|:---|
|TimeStamp|string|この型は時刻を格納します。協定世界時（UTC）を使用します。<br>フォーマットはYYYYMMDDhhmmss.nnnnnnnnnです。<br>YYYY,MM,DD,hh,mm,ss,およびnnnnnnnnnは，それぞれ西暦，月，日，時，分，秒，およびナノ秒を表します。<br>小数点以下の時刻については省略できます。<br>また，小数点以下には9桁までしか値を挿入できません。<br>小数点以下を省略した場合，または小数点以下が9桁未満の場合には余った桁部に0が挿入されます。<br>(Ex.100 -> 100.000000000, 100.1234 -> 100.123400000)|
|Boolean|true, false|この型は真偽値を格納します。true or falseを指定し，その値の真偽を示します。|
|Number|number|number型の値を格納します。この型を指定された要素では，特記しない限り，その値を0~2147483647の範囲に収める必要があります。|
|String255|string|string型の値を格納します。この型を指定された要素では，その文字数を255文字以内にしてください。|
|URI2047|string|string型の値を格納します。この型を指定された要素では，その文字数を2047文字以内にしてください(MUST)。また，RFC3986,RFC6874で定義されるURIに適合し，2047オクテット以下であるべきです(SHOULD)。|
|String32767|string|string型の値を格納します。この型を指定された要素では，その文字数を32767文字以内にしてください。|

文字数は最終的な表現にかかわらず、NFC正規化後のUTF-32コードポイント数で数えます(MUST)。

## 起動時の動作について

 - HAPやHatoholサーバーの起動，または再起動直後に自身と接続相手が使用可能なプロシージャの一覧をexchangeProfileプロシージャを用いて交換します。使用可能ではないプロシージャを呼び出したとき、通信先はエラーオブジェクトとともに-32601のエラーコードを返します。このエラーコードはJSON-RPC 2.0の仕様書の中で、"The method does not exist / is not available."と意味づけられています。

## プロシージャ

 - 「M/O」はそのプロシージャがMandatory(必須)かOptional(任意)であるかを表します。Mandatoryであるプロシージャは実装を省略できません。
 - exchangeProfileプロシージャ同士によるプロフィール交換が完了していない状態で，他のプロシージャによるリクエストや通知が届いた場合は，[putResult](#user-content-putresult)形式レスポンスのresultオブジェクトの内容を”FAILURE”とし，通信相手に返答する必要があります。

### Hatoholサーバーに実装するプロシージャ

|プロシージャ名|解説|タイプ|M/O|
|:-------------|:---|:-----|:-:|
|[exchangeProfile](#user-content-exchangeprofilemethod)|HAPが実装しているプロシージャ一覧とHAPの名前を受け取り，そのレスポンスとして自身が実装しているプロシージャ一覧と自身の名前を返します。|method|M|
|[getMonitoringServerInfo](#user-content-getmonitoringserverinfomethod)|HAPとの接続情報やポーリング間隔等をHAPに返します。|method|M|
|[getLastInfo](#user-content-getlastinfomethod)|リクエストで指定された要素の最新情報をHAPに返します。|method|M|
|[putItems](#user-content-putitemsmethod)|HAPが監視しているアイテム一覧を受け取ります。|method|O|
|[putHistory](#user-content-puthistorymethod)|HAPが監視している各アイテムのヒストリーを受け取ります。|method|O|
|[putHosts](#user-content-puthostsmethod)|HAPからホスト一覧を受け取り，更新します。|method|O|
|[putHostGroups](#user-content-puthostgroupsmethod)|HAPからホストグループ一覧を受け取り，更新します。|method|O|
|[putHostGroupMembership](#user-content-puthostgroupmembershipmethod)|HAPからホストグループ所属情報を受け取り，更新します。|method|O|
|[putTriggers](#user-content-puttriggersmethod)|HAPからトリガー情報を受け取り，更新します。|method|O|
|[putEvents](#user-content-puteventsmethod)|HAPからイベント情報を受け取り，更新します。|method|O|
|[putHostParents](#user-content-puthostparentsmethod)|HAPが監視しているホスト同士の親子関係を更新します。|method|O|
|[putArmInfo](#user-content-putarminfomethod)|HAPの接続ステータスを更新します。|method|M|

### HAPに実装するプロシージャ

|プロシージャ名|解説|タイプ|M/O|
|:-------------|:---|:-----|:-:|
|[exchangeProfile](#user-content-exchangeprofilemethod)|Hatoholサーバーが実装しているプロシージャ一覧とHatoholサーバーの名前を受け取り，そのレスポンスとして自身が実装しているプロシージャ一覧と自身の名前を返します。|method|M|
|[fetchItems](#user-content-fetchitemsmethod)|Hatoholサーバーからのアイテム取得リクエストを受け入れます。|method|O|
|[fetchHistory](#user-content-fetchhistorymethod)|Hatoholサーバーからのヒストリー取得リクエストを受け入れます。|method|O|
|[fetchTriggers](#user-content-fetchtriggersmethod)|Hatoholサーバーからのトリガー取得リクエストを受け入れます。|method|O|
|[fetchEvents](#user-content-fetcheventsmethod)|Hatoholサーバーからのイベント取得リクエストを受け入れます。|method|O|
|[updateMonitoringServerInfo](#user-content-updatemonitoringserverinfonotification)|Hatoholサーバーとの接続情報やポーリング間隔情報を通知として受け取ります。|notification|M|

### exchangeProfile(method)

 - Hatoholサーバー，HAP両者に共通するプロシージャです。主に初期起動時，再起動時に使用することを標準的な動作とします。詳細については[起動時の動作について](#user-content-起動時の動作について)をご覧ください。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|procedures|String255配列|M|-|送信元が使用可能なプロシージャ一覧です。|
|name      |String255    |M|-|送信元のプロセス名です。接続完了の旨を伝えるログなどに利用されます。|

```json
{
  "id": 1,
  "params": {
    "name": "exampleName",
    "procedures": [
      "getMonitoringServerInfo",
      "getLastInfo",
      "putItems",
      "putArmInfo",
      "fetchItems"
    ]
  },
  "method": "exchangeProfile",
  "jsonrpc": "2.0"
}
```

***リザルト(result)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|procedures|String255配列|M|-|送信先が使用可能なプロシージャ一覧です。|
|name      |String255    |M|-|送信先のプロセス名です。接続完了の旨を伝えるログなどに利用されます。|

```json
{
  "id": 1,
  "result": {
    "name": "exampleName",
    "procedures": [
      "getMonitoringServerInfo",
      "getLastInfo",
      "putItems",
      "putHistory",
      "putHosts",
      "putHostGroups",
      "putHostGroupMembership",
      "putTriggers",
      "putEvents",
      "putHostParent",
      "putArmInfo"
    ]
  },
  "jsonrpc": 2
}
```

### getMonitoringServerInfo(method)

 - ポーリング時間毎にHatoholサーバーに自身の接続情報やポーリング間隔等を問い合わせることを標準的な動作としますが，任意のタイミングで問い合わせることもできます。

***リクエスト(params)***

 - getMonitoringServerInfoメソッドには引数が存在しません。paramsオブジェクトの値を空文字にしてリクエストを送信してください。

```json
{
  "id": 1,
  "params": "",
  "method": "getMonitoringServerInfo",
  "jsonrpc": "2.0"
}
```

***リザルト(result)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|serverId          |Number     |M|-|監視サーバーのサーバーIDです。|
|url               |URI2047    |M|-|監視サーバーのURL [[解説](#user-content-servertype)]です。|
|type              |string     |M|-|監視サーバーの種類 [[一覧](#user-content-servertype)]です。|
|nickName          |String255  |M|-|監視サーバーのニックネームです。|
|userName          |String255  |M|-|監視サーバーのユーザーネームです。|
|password          |String255  |M|-|監視サーバーのパスワードです。|
|pollingIntervalSec|Number     |M|-|ポーリングを行う間隔です。|
|retryIntervalSec  |Number     |M|-|ポーリングが失敗した場合，リトライを行うまでの間隔です。|
|extendedInfo      |String32767|M|-|プラグイン固有の情報を格納することができます。|

```json
{
  "id": 1,
  "result": {
    "extendedInfo": "exampleExtraInfo",
    "serverId": 1,
    "url": "http://example.com:80",
    "type": "12345678-9abc-def0-1234-567891abcdef",
    "nickName": "exampleName",
    "userName": "Admin",
    "password": "examplePass",
    "pollingIntervalSec": 30,
    "retryIntervalSec": 10
  },
  "jsonrpc": "2.0"
}
```

### getLastInfo(method)

 - putプロシージャの呼び出し前に，Hatoholサーバーに送信，保存されたlastInfo情報を要求，取得します。取得したlastInfoを用い，前回までに送信したデータと現在のデータの差分をHatoholサーバーに送信します。
 - 初回起動時など，HatoholサーバーにlastInfoが保存されていない場合は，resultオブジェクトの値は空文字として返ってきます。

***リクエスト(params)***

|paramsオブジェクトの値|型 |M/O|デフォルト値|解説|
|:---------------------|:--|:-:|:----------:|:---|
|指定要素|String255|M|-|どの種類のlastInfoが必要かを指定する必要があります。以下の表を参照してください。|

|paramsオブジェクトの値一覧|解説|
|:-------------------------|:---|
|"host"               |ホストの最新情報を指定します。|
|"hostGroup"          |ホストグループの最新情報を指定します。|
|"hostGroupMembership"|ホストグループの所属情報の最新情報を指定します。|
|"trigger"            |トリガーの最新情報を指定します。|
|"event"              |イベントの最新情報を指定します。|
|"hostParent"         |ホストの親子関係の最新情報を指定します。|

```json
{
  "id": 1,
  "params": "trigger",
  "method": "getLastInfo",
  "jsonrpc": "2.0"
}
```

***リザルト(result)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|最新情報|String255|M|-|Hatoholサーバーに保存されている指定した要素の最新情報です。|

```json
{
  "id": 1,
  "result": "201504011349",
  "jsonrpc": "2.0"
}
```
この例ではlastInfoとしてタイムスタンプが返ってきています。


### putItems(method)

 - Hatoholサーバーとの接続完了時，または[fetchItems](#user-content-fetchitemsmethod)プロシージャのリクエストをHatoholサーバーから受け取った時に全てのアイテム情報をHatoholサーバーへ送信することを標準動作とします。Hatoholサーバーの負荷が高くなることが危惧されるため，任意のタイミングで使用することはできません。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|items  |object   |M|-|アイテム情報を格納するオブジェクトを配置します。詳細は次のテーブルを確認してください。|
|fetchId|String255|O|-|Hatoholサーバーから送られたどのリクエストに対するレスポンスであるかを示すIDです。fetchItemsのparams内のfetchIdの値をここに入れてください。|
|isLast   |Boolean  |O|-|リクエストを分割して送信する場合に使用します。分割したリクエストの最後か否か記してください。|
|serialId |Number   |O|-|リクエストを分割して送信する場合に使用します。分割したリクエストの何番目かを記してください。|
|requestId|Number   |O|-|リクエストを分割して送信する場合に使用します。分割したリクエスト内で同じ値を使用してください。|


***itemsオブジェクト***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|itemId       |String255    |M|-|アイテムのIDです。|
|hostId       |String255    |M|-|アイテムが所属するホストのIDです。|
|brief        |String255    |M|-|アイテムの概要です。|
|lastValueTime|TimeStamp    |M|-|アイテムが最後に更新された時刻です。|
|lastValue    |String255    |M|-|アイテムが最後に更新された際の値です。|
|itemGroupName|String255配列|M|-|アイテムが所属しているグループ名です。|
|unit         |String255    |M|-|valueの単位です。|

```json
{
  "id": 1,
  "params": {
    "fetchId": "1",
    "items": [
      {
        "unit": "example unit",
        "itemGroupName": ["example name"],
        "lastValue": "example value",
        "lastValueTime": "20150410175500",
        "brief": "example brief",
        "hostId": "1",
        "itemId": "1"
      },
      {
        "unit": "example unit",
        "itemGroupName": ["example name", "network", "building-E1"],
        "lastValue": "example value",
        "lastValueTime": "201504101755",
        "brief": "example brief",
        "hostId": "1",
        "itemId": "2"
      }
    ]
  },
  "method": "putItems",
  "jsonrpc": "2.0"
}
```

***リザルト(result)***

 - 送信したデータが正常に更新されたかどうかをresultオブジェクトで受け取ります。受け取る値については[[一覧](#user-content-putresult)]をご覧ください。

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### putHistory(method)

 - [fetchHistory](#user-content-fetchhistorymethod)プロシージャをHatoholサーバーから受け取った際に，条件にマッチするヒストリーをHatoholサーバーに送信します。Hatoholサーバーの負荷が高くなることが危惧されるため，任意のタイミングで使用することはできません。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|itemId    |String255 |M|-|取得するイベントのIDです。|
|samples   |object配列|M|-|ヒストリー情報を構成するサンプルの配列です。詳細は次のテーブルを確認してください。サンプルは、時刻の昇順に並んでいる必要があります。|
|fetchId   |String255 |O|-|Hatoholサーバーから送られたどのリクエストに対するレスポンスであるかを示すIDです。fetchHistoryのparams内のfetchIdの値をここに入れてください。|
|isLast   |Boolean  |O|-|リクエストを分割して送信する場合に使用します。分割したリクエストの最後か否か記してください。|
|serialId |Number   |O|-|リクエストを分割して送信する場合に使用します。分割したリクエストの何番目かを記してください。|
|requestId|Number   |O|-|リクエストを分割して送信する場合に使用します。分割したリクエスト内で同じ値を使用してください。|

***samplesオブジェクト***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|value |String255|M|-|time時点でのアイテムの値です。|
|time  |TimeStamp|M|-|valueが記録された時刻です。|

```json
{
  "id": 1,
  "params": {
    "fetchId": "1",
    "samples": [
      {
        "time": "20150323113000",
        "value": "exampleValue"
      },
      {
        "time": "201503231130",
        "value": "exampleValue2"
      }
    ],
    "itemId": "1"
  },
  "method": "putHistory",
  "jsonrpc": "2.0"
}
```

***リザルト(result)***

 - 送信したデータが正常に受け取られたかどうかをresultオブジェクトで受け取ります。受け取る値については[[一覧](#user-content-putresult)]をご覧ください。

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### putHosts(method)

 - Hatoholサーバーとの接続完了時，またはHAPが内部的に保存している登録ホスト情報が変更された際は,"ALL"オプションを用いて全てのホスト情報をHatoholサーバーに送信します。
 - "UPDATE"オプションを用いた場合は[getLastInfo](#user-content-getlastinfomethod)プロシージャ，またはHAP自身から呼び出したlastInfoを基に，その時点から現時点までに追加されたホストをHatoholサーバーに送信します。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|hosts       |object配列|M|-|ホスト情報を格納するオブジェクトを配置します。詳細は次のテーブルを確認してください。|
|updateType|string    |M|-|送信オプション[[一覧](#user-content-updatetype)]の中から状況に応じた送信オプションを選択してください。|
|lastInfo    |String32767 |O|-|最後に送信したホストの情報を送信する。この情報が[getLastInfo](#user-content-getlastinfomethod)の返り値になる。|
|isLast   |Boolean  |O|-|リクエストを分割して送信する場合に使用します。分割したリクエストの最後か否か記してください。|
|serialId |Number   |O|-|リクエストを分割して送信する場合に使用します。分割したリクエストの何番目かを記してください。|
|requestId|Number   |O|-|リクエストを分割して送信する場合に使用します。分割したリクエスト内で同じ値を使用してください。|

***hostsオブジェクト***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|hostId  |String255|M|-|監視サーバーが監視しているホストIDです。|
|hostName|String255|M|-|監視サーバーが監視しているホスト名です。|

```json
{
  "id": 1,
  "params": {
    "lastInfo": "201504091052",
    "updateType": "UPDATED",
    "hosts": [
      {
        "hostName": "exampleHostName1",
        "hostId": "1"
      }
    ]
  },
  "method": "putHosts",
  "jsonrpc": "2.0"
}
```

***リザルト(result)***

 - 送信したデータが正常に更新されたかどうかをresultオブジェクトで受け取ります。受け取る値については[[一覧](#user-content-putresult)]をご覧ください。

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### putHostGroups(method)

 - Hatoholサーバーとの接続完了時，またはHAPが内部的に保存している登録ホスト情報が変更された際は"ALL"オプションを用い，全てのホストグループ情報をHatoholサーバーに送信します。
 - "UPDATE"オプションを用いた場合は[getLastInfo](#user-content-getlastinfomethod)プロシージャ，またはHAP自身から呼び出したlastInfoを基に，その時点から現時点までに追加されたホストグループをHatoholサーバーに送信します。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|hostGroups  |object配列|M|-|ホストグループ情報を格納するオブジェクトを配置します。詳細は次のテーブルを確認してください。|
|updateType|string    |M|-|送信オプション[[一覧](#user-content-updatetype)]の中から状況に応じた送信オプションを選択してください。|
|lastInfo    |String32767|O|-|最後に送信したホストグループの情報を送信する。この情報が[getLastInfo](#user-content-getlastinfomethod)の返り値になります。|
|isLast   |Boolean  |O|-|リクエストを分割して送信する場合に使用します。分割したリクエストの最後か否か記してください。|
|serialId |Number   |O|-|リクエストを分割して送信する場合に使用します。分割したリクエストの何番目かを記してください。|
|requestId|Number   |O|-|リクエストを分割して送信する場合に使用します。分割したリクエスト内で同じ値を使用してください。|

***hostGroupsオブジェクト***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|groupId  |String255|M|-|ホストグループのIDです。|
|groupName|String255|M|-|グループIDに対応したホストグループの名前です。|

```json
{
  "id": 1,
  "params": {
    "lastInfo": "201504091049",
    "updateType": "ALL",
    "hostGroups": [
      {
        "groupName": "Group2",
        "groupId": "1"
      }
    ]
  },
  "method": "putHostGroups",
  "jsonrpc": "2.0"
}
```

***リザルト(result)***

 - 送信したデータが正常に更新されたかどうかをresultオブジェクトで受け取ります。受け取る値については[[一覧](#user-content-putresult)]をご覧ください。

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### putHostGroupMembership(method)

 - Hatoholサーバーとの接続完了時，またはHAPが内部的に保存している登録ホスト情報が変更された際は"ALL"オプションを用い，全てのホストグループ所属情報をHatoholサーバーに送信します。
 - "UPDATE"オプションを用いた場合は[getLastInfo](#user-content-getlastinfomethod)プロシージャ，またはHAP自身から呼び出したlastInfoを基に，その時点から現時点までに追加されたホストグループ所属情報をHatoholサーバーに送信します。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|hostGroupMembership|object配列|M|-|ホストグループ所属情報を格納するオブジェクトを配置します。詳細は次のテーブルを確認してください。|
|updateType       |string    |M|-|送信オプション[[一覧](#user-content-updatetype)]の中から状況に応じた送信オプションを選択してください。|
|lastInfo           |String32767|O|-|最後に送信したホストグループ所属情報の情報を送信する。この情報が[getLastInfo](#user-content-getlastinfomethod)の返り値になる。|
|isLast   |Boolean  |O|-|リクエストを分割して送信する場合に使用します。分割したリクエストの最後か否か記してください。|
|serialId |Number   |O|-|リクエストを分割して送信する場合に使用します。分割したリクエストの何番目かを記してください。|
|requestId|Number   |O|-|リクエストを分割して送信する場合に使用します。分割したリクエスト内で同じ値を使用してください。|

***hostGroupMembershipオブジェクト***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|hostId  |String255    |M|-|ホストのIDです。|
|groupIds|String255配列|M|-|ホストグループのIDです。|

```json
{
  "id": 1,
  "params": {
    "updateType": "ALL",
    "lastInfo": "201504091056",
    "hostGroupMembership": [
      {
        "groupIds": [
          "1",
          "2",
          "5"
        ],
        "hostId": "1"
      }
    ]
  },
  "method": "putHostGroupMembership",
  "jsonrpc": "2.0"
}
```

***リザルト(result)***

 - 送信したデータが正常に更新されたかどうかをresultオブジェクトで受け取ります。受け取る値については[[一覧](#user-content-putresult)]をご覧ください。

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### putTriggers(method)

[getLastInfo](#user-content-getlastinfomethod)を用いて取得，またはHAP自身が保管している最新トリガー情報を基に，そのトリガーから現時点までに更新されたトリガーをHatoholサーバーに送信するか，全てのトリガーを送信します。
 - Hatoholサーバーとの接続完了時，fetchTriggersプロシージャによる要求があった際は"ALL"オプションを用い，全てのトリガーをHatoholサーバーに送信します。
 - "UPDATE"オプションを用いた場合は[getLastInfo](#user-content-getlastinfomethod)プロシージャ，またはHAP自身から呼び出したlastInfoを基に，その時点から現時点までに更新，追加されたトリガーをHatoholサーバーに送信します。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|triggers    |object配列|M|-|トリガー情報を格納するオブジェクトを配置します。詳細は次のテーブルを確認してください。|
|updateType|String255 |M|-|送信オプション[[一覧](#user-content-updatetype)]の中から状況に応じた送信オプションを選択してください。|
|lastInfo    |String32767|O|-|最新トリガーの情報を送信する。この情報が[getLastInfo](#user-content-getlastinfomethod)の返り値になる。|
|fetchId     |String255 |O|-|Hatoholサーバーから送られたどのリクエストに対するレスポンスであるかを示すIDです。fetchTriggersによるリクエストを受けた場合にのみ、fetchTriggersのparams内のfetchIdの値をここに入れてください。|
|isLast   |Boolean  |O|-|リクエストを分割して送信する場合に使用します。分割したリクエストの最後か否か記してください。|
|serialId |Number   |O|-|リクエストを分割して送信する場合に使用します。分割したリクエストの何番目かを記してください。|
|requestId|Number   |O|-|リクエストを分割して送信する場合に使用します。分割したリクエスト内で同じ値を使用してください。|

***triggersオブジェクト***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|triggerId     |String255  |M|-|トリガーのID。HAP自身のトリガーを送信する場合は，トリガーIDとホストIDを"_SELF_"と記述することで送信したトリガーをSELFトリガー扱いにできます。|
|status        |string     |M|-|トリガーのステータス [[一覧](#user-content-triggerstatus)]です。|
|severity      |string     |M|-|トリガーの深刻度 [[一覧](#user-content-triggerseverity)]です。|
|lastChangeTime|TimeStamp  |M|-|トリガーが最後に更新された時刻です。|
|hostId        |String255  |M|-|トリガーが所属するホストIDです。|
|hostName      |String255  |M|-|トリガーが所属するサーバーのホスト名です。|
|brief         |String255  |M|-|トリガーの概要です。|
|extendedInfo  |String32767|M|-|上記の情報以外の必要な情報。主にWebUI上にデータを表示する際に用いられます。|

```json
{
  "id": 1,
  "params": {
    "triggers": [
      {
        "extendedInfo": "sample extended info",
        "brief": "example brief",
        "hostName": "exampleName",
        "hostId": "1",
        "lastChangeTime": "20150323175800",
        "severity": "INFO",
        "status": "OK",
        "triggerId": "1"
      }
    ],
    "fetchId": "1",
    "lastInfo": "201504061606",
    "updateType": "UPDATED"
  },
  "method": "putTriggers",
  "jsonrpc": "2.0"
}
```

***リザルト(result)***

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### putEvents(method)

 - 一度に送信できるイベント数は1000件までです。1000件を越える場合は，複数回に分けて送信してください。
 - イベントIDが重複したイベントを送信することは認められてします。しかし，元から存在するイベントと送信するイベントのどちらが優先されるかは未定義です。
 - 自発的にイベントを送信する動作とfetchEventsプロシージャに対するレスポンスとしてイベントを送信する動作の2つの動作が存在します。
 - 自発的にイベントを送信する場合は，まず[getLastInfo](#user-content-getlastinfomethod)のレスポンスや，HAP自身からlastInfoを取得します。取得したlastInfoで判別したイベントから現時点までに発生した差分のイベントをHatoholサーバーに送信します。初回通信時は，lastInfoの値が空文字であるため，接続以前に発生したイベントをすべて送信するか，何も送信しない選択がHAP作成者に委ねられています。
 - fetchEventsプロシージャのlastInfoで指定されたイベントより先にイベントが存在しない場合は，eventsオブジェクトの値を空配列にして送信してください。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|events     |object配列|M|-|イベント情報を格納するオブジェクトを配置します。詳細は次のテーブルを確認してください。|
|lastInfo   |String32767|O|-|イベントを送信する際，次回イベントを送信する際の基準となる情報を送信する。この情報が[getLastInfo](#user-content-getlastinfomethod)の返り値になる。しかし，mayMoreFlagの値がtrueとなっている場合，この値はHatoholのDBへは保存されずHatoholサーバープロセスに一時的に保存されます。|
|mayMoreFlag|Boolean   |O|-|fetchEventsプロシージャに対するレスポンスとしてputEventsプロシージャを用いる場合のみ，fetchIdと合わせてparamsに挿入してください。<br>指定された件数に満たない件数のイベントを送信し，送信すべきイベントがまだ残っている可能性がある場合に値をtrueとしてください。<br>この値をtrueにする場合，最低限イベントを1件は送信する必要があります。|
|fetchId    |String255 |O|-|Hatoholサーバーから送られたどのリクエストに対するレスポンスであるかを示すIDです。fetchEventsによるリクエストを受けた場合にのみ、fetchEventsのparams内のfetchIdの値をここに入れてください。|
|isLast   |Boolean  |O|-|リクエストを分割して送信する場合に使用します。分割したリクエストの最後か否か記してください。|
|serialId |Number   |O|-|リクエストを分割して送信する場合に使用します。分割したリクエストの何番目かを記してください。|
|requestId|Number   |O|-|リクエストを分割して送信する場合に使用します。分割したリクエスト内で同じ値を使用してください。|

***eventsオブジェクト***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|eventId     |String255  |M|-|イベントのIDです。|
|time        |TimeStamp  |M|-|イベントが発生した時刻です。|
|type        |string     |M|-|イベントのタイプ [[一覧](#user-content-eventtype)]です。|
|triggerId   |String255  |O|-|このイベントを発火させたトリガーID。トリガーとイベントを関連付けないことも可能なため，必須ではありません。|
|status      |string     |O|-|トリガーのステータス [[一覧](#user-content-triggerstatus)]です。|
|severity    |string     |O|-|トリガーの深刻度 [[一覧](#user-content-triggerseverity)]です。|
|hostId      |String255  |O|-|イベントが発生したホストのIDです。|
|hostName    |String255  |O|-|イベントが発生したホストの名前です。|
|brief       |String255  |M|-|イベントの説明。Web上に表示される情報です。|
|extendedInfo|String32767|O|-|briefには書いていない追加の情報を記述できます。|

```json
{
  "id": 1,
  "params": {
    "fetchId": "1",
    "mayMoreFlag": true,
    "lastInfo": "201504011759",
    "events": [
      {
        "extendedInfo": "sampel extended info",
        "brief": "example brief",
        "eventId": "1",
        "time": "20150323151300",
        "type": "GOOD",
        "triggerId": 2,
        "status": "OK",
        "severity": "INFO",
        "hostId": 3,
        "hostName": "exampleName"
      }
    ]
  },
  "method": "putEvents",
  "jsonrpc": "2.0"
}
```

***リザルト(result)***

 - 送信したデータが正常に更新されたかどうかをresultオブジェクトで受け取ります。受け取る値については[[一覧](#user-content-putresult)]をご覧ください。

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### putHostParents(method)

 - Hatoholサーバーとの接続完了時は"ALL"オプションを用い，全てのホストの親子関係をHatoholサーバーに送信します。
 - "UPDATE"オプションを用いた場合は[getLastInfo](#user-content-getlastinfomethod)プロシージャ，またはHAP自身から呼び出したlastInfoを基に，その時点から現時点までに追加されたホストの親子関係をHatoholサーバーに送信します。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|;
|hostParents  |object配列|M|-|ホストの親子関係を格納するオブジェクトを配置します。詳細は次のテーブルを確認してください。|
|updateType|String255 |M|-|送信オプション[[一覧](#user-content-updatetype)]の中から状況に応じた送信オプションを選択してください。|
|lastInfo    |String32767|O|-|最後に送信したホストグループ所属情報の情報を送信する。この情報が[getLastInfo](#user-content-getlastinfomethod)の返り値になる。|
|isLast   |Boolean  |O|-|リクエストを分割して送信する場合に使用します。分割したリクエストの最後か否か記してください。|
|serialId |Number   |O|-|リクエストを分割して送信する場合に使用します。分割したリクエストの何番目かを記してください。|
|requestId|Number   |O|-|リクエストを分割して送信する場合に使用します。分割したリクエスト内で同じ値を使用してください。|

***hostParentsオブジェクト***

 - ホストの親子関係を削除する場合は親ホストIDの値を空文字にすることで，送信した子ホストIDの親子関係をHatoholサーバーから削除することができます。

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|childHostId |String255|M|-|子ホストのIDです。|
|parentHostId|String255|M|-|親ホストのIDです。|

```json
{
  "id": 1,
  "params": {
    "lastInfo": "201504152246",
    "updateType": "ALL",
    "hostParents": [
      {
        "parentHostId": "10",
        "childHostId": "12"
      },
      {
        "parentHostId": "20",
        "childHostId": "11"
      }
    ]
  },
  "method": "putHostParent",
  "jsonrpc": "2.0"
}
```

***リザルト(result)***

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### putArmInfo(method)

HostやTrigger，Event情報の送信処理が行われるたびにHatoholサーバーに送信することを標準的な動作としますが，任意に送信してもかまいません。最小間隔は１秒（MUST），最大間隔は[getMonitoringServerInfo](#user-content-getmonitoringserverinfomethod)や[updateMonitoringServerInfo](#user-content-updatemonitoringserverinfonotification)で取得したポーリング時間の2倍（SHOULD）とします。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|lastStatus         |string   |M|-|最新のポーリング結果 [[一覧](#user-content-arminfostatus)]です。|
|failureReason      |String255|M|-|情報取得が失敗した理由です。|
|lastSuccessTime    |TimeStamp|M|-|最後に情報取得が成功した時刻です。一度も成功していない場合は空文字列をセットして下さい。|
|lastFailureTime    |TimeStamp|M|-|最後に情報取得が失敗した時刻です。一度も失敗していない場合は空文字列をセットして下さい。|
|numSuccess         |Number   |M|-|HAPが起動してから情報取得に成功した回数です。|
|numFailure         |Number   |M|-|HAPが起動してから情報取得に失敗した回数です。|

```json
{
  "id": 1,
  "params": {
    "numFailure": 10,
    "numSuccess": 165,
    "lastFailureTime": "20150313161500",
    "lastSuccessTime": "20150313161100",
    "failureReason": "Example reason",
    "lastStatus": "INIT"
  },
  "method": "putArmInfo",
  "jsonrpc": "2.0"
}
```

***リザルト(result)***

 - 送信したデータが正常に更新されたかどうかをresultオブジェクトで受け取ります。受け取る値については[[一覧](#user-content-putresult)]をご覧ください。

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### fetchItems(method)

Hatoholサーバーがアイテム情報を要求しているときにHAPに送信されます。このプロシージャを受け取った時，resultとしてリクエスト受け入れの成否を返す必要があります。その後，指定されたホストに属する全てのアイテムをputItemsプロシージャ[putItems](#user-content-putitemsmethod)を用いてHatoholサーバーに送信してください。その際，paramsのfetchIdオブジェクトの値を[putItems](#user-contents-putitemsmethod)に渡す必要があります。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|hostIds|String255配列|O|-|ホストを指定し取得するアイテムを限定します。hostIdsがセットされていない場合は全てのホストが対象となります。|
|fetchId|String255    |M|-|putItemsプロシージャで使用します。そのputItemsプロシージャがどのfetchItemsプロシージャによる要求に対応したものかをHatoholサーバーが識別するために必要です。|

```json
{
  "id": 1,
  "params": {
    "fetchId": "1",
    "hostIds": [
      "1",
      "2",
      "3"
    ]
  },
  "method": "fetchItems",
  "jsonrpc": "2.0"
}
```

***リザルト(result)***

 - リクエストの受け入れ成否をresultオブジェクトとしてHatoholサーバーに返す必要があります。返す値については[[一覧](#user-content-fetchresult)]をご覧ください。

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### fetchHistory(method)

 - このプロシージャは，Hatoholサーバーがヒストリーを要求しているときにHAPに送信されます。HAPはレスポンスとしてリクエスト受け入れの成否を返す必要があります。その後，指定条件に当てはまるヒストリーを[putHistoryプロシージャ](#user-content-puthistorymethod)を用いてHatoholサーバーに送信してください。その際，fetchHistoryプロシージャのparams内にあるfetchIdの値をputHistoryプロシージャに渡す必要があります。
 - paramsオブジェクト内にあるbeginTime，endTimeはbeginTime以上，endTime以下の条件に当てはまるHistory取得することを想定しています。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|hostId    |String255|M|-|ヒストリーのアイテムが所属しているホストIDです。|
|itemId    |String255|M|-|ヒストリーのアイテムIDです。|
|beginTime |TimeStamp|M|-|ヒストリー取得域の始点時刻を指定します。|
|endTime   |TimeStamp|M|-|ヒストリー取得域の終点時刻を指定します。|
|fetchId   |String255|M|-|putHistoryプロシージャで使用します。そのputHistoryプロシージャがどのfetchHistoryプロシージャによる要求に対応したものかをHatoholサーバーが識別するために必要です。|

```json
{
  "id": 1,
  "params": {
    "fetchId": 1,
    "beginTime": "20150323151300",
    "itemId": 1,
    "hostId": "1"
  },
  "method": "fetchHistory",
  "jsonrpc": "2.0"
}
```

***リザルト(result)***

 - リクエストの受け入れ成否をresultオブジェクトとしてHatoholサーバーに返す必要があります。返す値については[[一覧](#user-content-fetchresult)]をご覧ください。

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### fetchTriggers(method)

 - このプロシージャは，Hatoholサーバーがトリガー情報を要求しているときにHAPに送信されます。HAPはレスポンスとしてリクエスト受け入れの成否を返す必要があります。その後，[putTriggersプロシージャ](#user-content-puttriggersmethod)の"ALL"オプションを用いて指定されたホストに属する全てのトリガーを送信してください。その際，fetchIdとhostIdsの値をputTriggersプロシージャに渡す必要があります。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|hostIds|String255配列|O|-|ホストを指定し取得するトリガーを限定します。hostIdsがセットされていない場合は全てのホストが対象となります。|
|fetchId|String255    |M|-|putTriggersプロシージャで使用します。そのputTriggersプロシージャがどのfetchTriggersプロシージャによる要求に対応したものかをHatoholサーバーが識別するために必要です。|

```json
{
  "jsonrpc": "2.0",
  "method": "fetchTriggers",
  "params": {
    "hostIds": [
      "1",
      "2"
    ],
    "fetchId": "1"
  },
  "id": 1
}
```

***リザルト(result)***

 - リクエストの受け入れ成否をresultオブジェクトとしてHatoholサーバーに返す必要があります。返す値については[[一覧](#user-content-fetchresult)]をご覧ください。

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### fetchEvents(method)

 - 指定したイベントIDから昇順，または降順で指定した件数のイベントを取得するリクエストが送信されます。HAPはレスポンスとしてリクエスト受け入れの成否を返す必要があります。その後，指定された条件のイベントを[putEventsプロシージャ](#user-content-puteventsmethod)を用いて送信してください。その際，fetchEventsプロシージャのparamsオブジェクト内にあるfetchIdの値をputEventsプロシージャに渡す必要があります。
 - 指定できる取得件数の最大値は1000です。それより大きい件数を取得する場合、複数回のリクエストに分割してください。

***リクエスト(params)***

|オブジェクトの名前|型 |M/O|デフォルト値|解説|
|:-----------------|:--|:-:|:----------:|:---|
|lastInfo |String32767|M|-|基準となるイベントの情報です。|
|count    |Number   |M|-|取得するイベント件数です。最大は1000件です。|
|direction|String255|M|-|"ASC"（指定したIDより新しいイベント）または”DESC”(指定したIDより古いイベント)を選択します。|
|fetchId  |String255|M|-|putEventsプロシージャで使用します。そのputEventsプロシージャがどのfetchEventsプロシージャによる要求に対応したものかどうかをHatoholサーバーが識別するために必要です。|

```json
{
  "id": 1,
  "params": {
    "direction": "ASC",
    "count": 1000,
    "lastInfo": "10",
    "fetchId": "1"
  },
  "method": "fetchEvents",
  "jsonrpc": "2.0"
}
```

***リザルト(result)***

 - リクエストの受け入れ成否をresultオブジェクトとしてHatoholサーバーに返す必要があります。返す値については[[一覧](#user-content-fetchresult)]をご覧ください。

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### updateMonitoringServerInfo(notification)

 - 監視サーバー情報が更新された際に，Hatoholサーバーから送信される通知です。この通知を受け取った場合，プラグインをリスタートするなどして，各監視情報を更新することを標準的な動作とします。

***params***

 - paramsの内容は[getMonitoringServerInfo](#user-content-getmonitoringserverinfomethod)のreusltオブジェクトの内容と同一です。

```json
{
  "method": "updateMonitoringServerInfo",
  "params": {
    "extendedInfo": "exampleExtraInfo",
    "serverId": 1,
    "url": "http://example.com:80",
    "type": "12345678-9abc-def0-1234-567891abcdef",
    "nickName": "exampleName",
    "userName": "Admin",
    "password": "examplePass",
    "pollingIntervalSec": 30,
    "retryIntervalSec": 10
  },
  "jsonrpc": "2.0"
}
```

## 表

### armInfoStatus

|ステータス|解説|
|:---------|:---|
|"INIT"   |初期状態を表します。まだ通信を行っていない状態です。|
|"OK"     |通信に成功している状態を表します。|
|"NG"     |通信に失敗している状態を表します。|

### ServerType

 - HAPを識別する際に使用されている各サーバータイプのUUIDです。あなたがHAPを作成する場合，以下の表には存在しないUUIDを作成し，新たにサーバータイプを定義する必要があります。

|名前|UUID|
|:---|:---|
|Zabbix           |8e632c14-d1f7-11e4-8350-d43d7e3146fb|
|Nagios NDOUtils  |902d955c-d1f7-11e4-80f9-d43d7e3146fb|
|Nagios Livestatus|6f024e3e-a2cd-11e5-bfc7-d43d7e3146fb|
|Ceilometer       |aa25a332-d1f7-11e4-80b4-d43d7e3146fb|
|Fluentd          |d91ba1cb-a64a-4072-b2b8-2f91bcae1818|

### triggerSeverity

|深刻度|
|:---|
|"UNKNOWN"  |
|"INFO"     |
|"WARNING"  |
|"ERROR"    |
|"CRITICAL" |
|"EMERGENCY"|

### triggerStatus

|ステータス|解説|
|:---------|:---|
|"OK""    |通信に成功している状態を表します。|
|"NG"|通信に失敗している状態を表します。|
|"UNKNOWN"|状態不明を表しています。|

### updateType

|種類|解説|
|:---------|:---|
|"ALL"    |各データ全てを送信します。Hatoholサーバー内の古いデータを削除し，その後送信した全てのデータを登録します。|
|"UPDATED"|アップデートされたデータのみをHatoholサーバーに送信し，同一IDのデータは上書き，初出のデータは新規登録します。|

### eventType

|タイプ|解説|
|:-----|:--:|
|"GOOD"        |正常|
|"BAD"         |異常|
|"UNKNOWN"     |不明|
|"NOTIFICATION"|通知|

### putResult

 - putプロシージャをコールした際の更新の成否です。
 - 更新に失敗した際は，再度putプロシージャをコールするといった動作が標準的です。

|ステータス|解説|
|:---------|:---|
|"SUCCESS"|更新が正常に終了したことを表します。|
|"FAILURE" |更新が失敗したことを表します。|

 - また，putプロシージャのparamsの内容に不備があり，引数が間違っていたなどの理由でエラーを返す場合には，以下の例のようにJSON-RPC2.0で定義されているerrorオブジェクトを用いてエラーを返す必要があります。

 ```json
{
  "id": "1",
  "error": {
    "message": "Invalid params",
    "code": -32602
  },
  "jsonrpc": "2.0"
}
 ```

### fetchResult

 - fetchと名のついたプロシージャが呼び出された際に，そのリクエストを受け入れたかどうかの成否です。リクエストの受け入れに失敗した際は，再度リクエストするといった動作が標準的です。

|ステータス|解説|
|:---------|:---|
|"SUCCESS"|リクエストの受け入れに成功しました。|
|"ABBREV"|リクエストの間隔が近いため，リクエストの受け入れを省略しました。|
|"FAILURE"|リクエストの受け入れに失敗しました。|

 - また，fetchプロシージャのparamsの内容に不備があり，引数が間違っていたなどの理由でエラーを返す場合には，以下の例のようにJSON-RPC2.0で定義されているerrorオブジェクトを用いてエラーを返す必要があります。

 ```json
{
  "id": "1",
  "error": {
    "message": "Invalid params",
    "code": -32602
  },
  "jsonrpc": "2.0"
}
 ```

<!--
## 改版履歴
-->

## 連絡先
不明点、また改善の提案についてはHatoholコミュニティにお問い合わせください。[hatohol-users@lists.osdn.me]

## 著作権
Copyright (C)2015-2016 Project Hatohol
