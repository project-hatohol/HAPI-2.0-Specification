# Arm Plugin Interface 2.0 Specification(2015/04/16)

## Overview

Hatohol Arm Plugin Interface (HAPI) 2.0 is the protocol for information exchange between Hatohol server and Monitoring server plugins.
It is based on JSON-PRC, defines its own methods and types, and provides a typical operation sequence.

The following figure depicts the above overview.

![overview](../hapi_overview.png)

## Words

|words|description|
|:---|:---|
|SRV|Hatohol server|
|HAP|Hatohol monitoring server plugin|

"MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY" and "OPTIONAL" conform to [RFC2119](http://www.ietf.org/rfc/rfc2119.txt).

## Involved protocol

### Communication path 

Use Advanced Message Queuing Protocol (AMQP) 0.9.1 as a communication channel.
- AMQP 0.9.1: https://www.rabbitmq.com/resources/specs/amqp0-9-1.pdf

### JSON-RPC

Use JSON-RPC 2.0 as a basic protocol of information exchange.
- [JSON-RPC 2.0 (2013-01-04)](http://www.jsonrpc.org/specification)
- [JSON(RFC4627)](https://www.ietf.org/rfc/rfc4627.txt)

### Attention

- Character encoding MUST be UTF-8. Normalization form SHOULD be C. String MAY escape.
- Enough randomness SHOULD be needed from ID object that is used in request and response.
- Must not use batch request of JSON_RPC in HAPI2.0(MUST NOT).
- HAPI2.0 does not assume to generate and use AMQP queue name dynamically. The user need to dicide queue name that is used when contanct to Hatohol server. HAPI2.0 assume Hatohol server and plugin use the above queue name.

## Operating overview

The following sequence figure describes basic operations of procedures request and response that are exchanged between Hatohol server and Hatohol arm plugin. 

```

   SRV                                            HAP
    |
    |                                        Turn on HAP
    |                                              |
    |<----------exchangeProfile(Request)---------->|
    |<----------exchangeProfile(Response)--------->|
    |                                          |   |
    |                             Polling interval |
    |                                          |   |
    |<-----getMonitoringServerInfo(Request)--------|
    |------getMonitoringServerInfo(Response)------>|
    |                                              |
    |<-----------getLastInfo(Request)--------------|
    |------------getLastInfo(Response)------------>|
    |<------------putHosts(Request)----------------|
    |-------------putHosts(Response)-------------->|
    |                                              |
    |<-----------getLastInfo(Request)--------------|
    |------------getLastInfo(Response)------------>|
    |<---------putHostGroups(Request)--------------|
    |----------putHostGroups(Response)------------>|
    |                                              |
    |<-----------getLastInfo(Request)--------------|
    |------------getLastInfo(Response)------------>|
    |<-----putHostGroupMembership(Request)---------|
    |------putHostGroupMembership(Response)------->|
    |                                              |
    |<-----------getLastInfo(Request)--------------|
    |------------getLastInfo(Response)------------>|
    |<------------putTriggers(Request)-------------|
    |-------------putTriggers(Response)----------->|
    |                                              |
    |<-----------getLastInfo(Request)--------------|
    |------------getLastInfo(Response)------------>|
    |<------------putEvents(Request)---------------|
    |-------------putEvents(Response)------------->|
    |                                              |
    |<-----------getLastInfo(Request)--------------|
    |------------getLastInfo(Response)------------>|
    |<------------putHostParents(Request)----------|
    |-------------putHostParents(Response)-------->|
    |                                              |
    |<------------putArmInfo(Request)--------------|
    |-------------putArmInfo(Response)------------>|
    |                                          |   |
    |                             Polling interval |
    |                                          |   |
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    |                                              |
    |-----------fetchItems(Request)--------------->|
    |<----------fetchItems(Response)---------------|
    |<------------putItems(Request)----------------|
    |-------------putItems(Response)-------------->|
    |<------------finishPut(Request)---------------|
    |-------------finishPut(Responce)------------->|
    |                                              |
    |-----------fetchHistory(Request)------------->|
    |<----------fetchHistory(Response)-------------|
    |<------------putHistory(Request)--------------|
    |-------------putHistory(Response)------------>|
    |<------------finishPut(Request)---------------|
    |-------------finishPut(Responce)------------->|
    |                                              |
    |-----------fetchTriggers(Request)------------>|
    |<----------fetchTriggers(Response)------------|
    |<-----------putTriggers(Request)--------------|
    |                Use "ALL"option               |
    |-----------putTriggers(Response)------------->|
    |<------------finishPut(Request)---------------|
    |-------------finishPut(Responce)------------->|
    |                                              |
    |-------------fetchEvents(Request)------------>|
    |<------------fetchEvents(Response)------------|
    |<------------putEvents(Request)---------------|
    |-------------putEvents(Response)------------->|
    |<------------finishPut(Request)---------------|
    |-------------finishPut(Responce)------------->|
    |                                              |
    |--updateMonitoringServerInfo(notification)--->|
    |                                              |

```
## Data type

 - This section describes about data types that are defined for Hatohol. They are defined and used in JSON-RPC internally.

|Name|JSON type|description|
|:---|:---------|:---|
|TimeStamp|string|This type is used to store times of day. Time value MUST be stored in UTC. Format is YYYYMMDDhhmmss.nnnnnnnnn. YYYY, MM, DD, hh, mm, ss, and nnnnnnnnn each expresses the A.D., Month, Day, hour, minute, second, nano second. It can abbreviate nano second. If the above or nano second digits is less than nine, it will be inserted zero to fill digit(s)(Ex.100 -> 100.000000000, 100.1234 -> 100.123400000).|
|Boolean|true, false|This type is used to store boolean value. It can assign true or false to show authenticity of value.|
|Number|number|This type is used to store number value. It needs to be more than 0 and less than 2147483647.|
|String255|string|This type is used to store string value. It needs to be the number of characters less than 255 characters.|
|URI2047|string|This type is used to store string value. It MUST be the number of characters less than 2047. It should satisfy URI requirements which is defined in the RFC3986, RFC6874. And its octet size SHOULD be less than 2047.|
|String32767|string|This type is used to store string value. It needs to be the number of characters less than 32767 characters.|

In any case, the number of character MUST be counted by UTF-32 code point number after applying NFC nomalization.

## About start up operation

 - Immediatery after starting or restarting Hatohol server and HAP, they exchange available procedures information with exchangeProfile procedure. If you call a unavailable procedure, the connection partner returns error object with -32601 which means "The method does not exist / is not available." in the JSON-RPC 2.0 specification.

## Procedures

 - [M/O] indicates whether Mandatory or Optional. Mandatory procedure can not abbreviate to implement.
 - When not completing exchange profile with exchangeProfile procedure yet and call other procedures, it must return [putResult](#user-content-putresult) response that is inserted "FAILURE" to result object.

### Hatohol server procedures

|name|description|type|M/O|
|:-------------|:---|:-----|:-:|
|[exchangeProfile](#user-content-exchangeprofilemethod)|Receive view of procedures that is implemented in HAP and HAP name, after that return procedures view that is implemented in yourself and yourself name.|method|M|
|[getMonitoringServerInfo](#user-content-getmonitoringserverinfomethod)|Return connecting information with HAP and polling interval seconds to HAP.|method|M|
|[getLastInfo](#user-content-getlastinfomethod)|Return latest information that is designated by request.|method|M|
|[putItems](#user-content-putitemsmethod)|Receive items view that is monitored by HAP.|method|O|
|[putHistory](#user-content-puthistorymethod)|Receive history of each item that is monitored by HAP.|method|O|
|[putHosts](#user-content-puthostsmethod)|Receive hosts view from HAP and update it.|method|O|
|[putHostGroups](#user-content-puthostgroupsmethod)|Receive host groups view from HAP and update it.|method|O|
|[putHostGroupMembership](#user-content-puthostgroupmembershipmethod)|Receive host group membership from HAP and update it.|method|O|
|[putTriggers](#user-content-puttriggersmethod)|Receive triggers view from HAP and update it.|method|O|
|[putEvents](#user-content-puteventsmethod)|Receive events view from HAP and update it.|method|O|
|[putHostParents](#user-content-puthostparentsmethod)|Receive host parent relations view from HAP and update it.|method|O|
|[finishPut](#user-content-finishputmethod)|Notify to nothing put procedure that is responce of fetch procedure.|method|M|
|[putArmInfo](#user-content-putarminfomethod)|Receive connecting status of HAP and update it.|method|M|

### HAP procedures

|name|desscription|type|M/O|
|:-------------|:---|:-----|:-:|
|[exchangeProfile](#user-content-exchangeprofilemethod)|Receive view of procedures that is implemented in Hatohol server and Hatohol server name, after that return procedures view that is implemented in yourself and yourself name.|method|M|
|[fetchItems](#user-content-fetchitemsmethod)|Accept request to get item from Hatohol server.|method|O|
|[fetchHistory](#user-content-fetchhistorymethod)|Accept request to get history from Hatohol server.|method|O|
|[fetchTriggers](#user-content-fetchtriggersmethod)|Accept request to get trigger from Hatohol server.|method|O|
|[fetchEvents](#user-content-fetcheventsmethod)|Accept request to get event from Hatohol server.|method|O|
|[updateMonitoringServerInfo](#user-content-updatemonitoringserverinfomethod)|Receive connecting information with Hatohol server and polling interval seconds as a notification.|method|M|

### exchangeProfile(method)

Both of Hatohol server and HAP MUST have this procedure. This procedure is mainly used after starting and restarting. Please look at the [About start up operation](#user-content-about-start-up-content).

***Request(params)***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|procedures|String255 array|M|-|Usual procedures list of sender.|
|name      |String255    |M|-|Process name of sender. It is used in the log message and etc.|

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

***Result(result)***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|procedures|String255 array|M|-|Usual procedures list of destination.|
|name      |String255    |M|-|Process name of destination. It is used in the log message and etc.|

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

This procedure is mainly used to acquire Hatohol server itself connection and polling interval information. And this procedure can call at the arbitrary point of the time.

***Request(params)***

 getMonitoringServerInfo method does not have arguments. It should request with a empty string parameter.

```json
{
  "id": 1,
  "params": "",
  "method": "getMonitoringServerInfo",
  "jsonrpc": "2.0"
}
```

***Result(result)***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|serverId          |Number     |M|-|A Server ID of the monitoring server.|
|url               |URI2047    |M|-|A URL of monitoring server. [[Description](#user-content-servertype)]|
|type              |string     |M|-|A type of monitoring server. [[List](#user-content-servertype)]|
|nickName          |String255  |M|-|A nickname of monitoring server.|
|userName          |String255  |M|-|A user name of monitoring server.|
|password          |String255  |M|-|A pass word of monitoring server.|
|pollingIntervalSec|Number     |M|-|Polling interval seconds.|
|retryIntervalSec  |Number     |M|-|Retry interval seconds in case of fail polling.|
|extendedInfo      |String32767|M|-|This field can store an additional information.|

```json
{
  "id": 1,
  "result": {
    "serverId": 1,
    "url": "http://example.com:80",
    "type": "12345678-9abc-def0-1234-567891abcdef",
    "nickName": "exampleName",
    "userName": "Admin",
    "password": "examplePass",
    "pollingIntervalSec": 30,
    "retryIntervalSec": 10,
    "extendedInfo": "exampleExtraInfo"
  },
  "jsonrpc": "2.0"
}
```

### getLastInfo(method)

This procedure is used for getting lastInfo which is stored in a Hatohol server before calling put series procedures.
Its lastInfo is used for put series procedures to calculate changes current value from previous value.

No lastInfo in a Hatohol server, it SHALL return an empty string.

***Request(params)***

|params object value|Type |M/O|Default value|Description|
|:---------------------|:--|:-:|:----------:|:---|
|Designate element|String255|M|-|Need to select in the following elements list.|

|Value list|Description|
|:-------------------------|:---|
|"host"               |Select latest host information.|
|"hostGroup"          |Select latest host group information.|
|"hostGroupMembership"|Select latest host group membership information.|
|"trigger"            |Select latest trigger information.|
|"event"              |Select latest event information.|
|"hostParent"         |Select latest host parent information.|

```json
{
  "id": 1,
  "params": "trigger",
  "method": "getLastInfo",
  "jsonrpc": "2.0"
}
```

***Result(result)***

|Object value|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|latest information|String255|M|-|Designate element latest information of to be saved in Hatohol server.|

```json
{
  "id": 1,
  "result": "201504011349",
  "jsonrpc": "2.0"
}
```
In this example, callee receive a timestamp as a lastInfo.

### putItems(method)

Standard behavior is to send all item information to Hatohol server when complete connection to Hatohol server and get [fetchItems](#user-content-fetchitemsmethod) request from Hatohol server. There is a possibility that causes a high load on a Hatohol Server, so this procedure forbid to use at the arbitrary point of the time.

***Request(params)***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|items  |object   |M|-|This is an object that stores item information. In more detail, please refer the following table.|
|fetchId|String255|O|-|This is an ID which corresponds for request from Hatohol server. It should be contain fetchId only if it will be received fetchItems request.|

***items object***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|itemId       |String255    |M|-|ID of the item.|
|hostId       |String255    |M|-|Host ID of the the item belongs.|
|brief        |String255    |M|-|Overview of the item.|
|lastValueTime|TimeStamp    |M|-|Time of when the item was last updated.|
|lastValue    |String255    |M|-|Value of when the item was last updated.|
|itemGroupName|String255 array|M|-|Group name of the item belongs.|
|unit         |String255    |M|-|Unit of value.|

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

***Result(result)***

Caller receives result as a result object value whether sent data has been updated. Refer to the [[Values](#user-content-putresult)].

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### putHistory(method)

 - When receiving [fetchHistory](#user-content-fetchhistorymethod) procedure from Hatohol server, HAP returns matched history to conditions. This procedure has some possibility that burden to Hatohol server and can not use at the arbitrary point of time. 

***Request(params)***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|itemId    |String255 |M|-|Id of the got item|
|samples   |object array|M|-|This is sample array to configure history information. In more detail, please refer the following table. The samples is needed to sort by ascending order of time.|
|fetchId|String255|O|-|This is an ID that indicates whether response for request from Hatohol server. Insert fetchId value to this object, when receiving fetchHistory request.|

***samples object***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|value |String255|M|-|Item value at that time.|
|time  |TimeStamp|M|-|Time of recorded.|

```json
{
  "id": 1,
  "params": {
    "fetchId": "1",
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

***Result(result)***

Caller receives result as a result object value whether sent data has been updated. Refer to the [[Values](#user-content-putresult)].

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### putHosts(method)

Send all hosts information to Hatohol server to use "ALL" option when complete connection to Hatohol server and changed hosts information that saved in internal.

When using "UPDATE" option, send difference to use lastInfo that got using [getLastInfo](#user-content-getlastinfomethod) or saved in internal to Hatohol server.

***Request(params)***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|hosts       |object array|M|-|This is an object array that stores hosts information. In more detail, please refer the following table.|
|updateType|string    |M|-|Select sending option that match the situation from [[List](#user-content-updatetype)].|
|lastInfo    |String32767 |O|-|Insert to here host information of last. A result of [getLastInfo](#user-content-getlastinfomethod) is constructed from this information.|

***hosts object***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|hostId  |String255|M|-|Host Id of a host which monitered under monitoring server.|
|hostName|String255|M|-|Host name of a host which monitored under monitoring server.|

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

***Result(result)***

Caller receives result as a result object value whether sent data has been updated. Refer to the [[Values](#user-content-putresult)].

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### putHostGroups(method)

HAP sends all host groups information to Hatohol server to use "ALL" option when complete connection to Hatohol server and changed hosts information that stored in internal.

When using "UPDATE" option, send difference to use lastInfo that got using [getLastInfo](#user-content-getlastinfomethod) or stored in internal to Hatohol server.

***Request(params)***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|hostGroups  |object array|M|-|This is an object array that stores host groups information. In more detail, please refer the following table.|
|updateType|string    |M|-|Select sending option that match the situation from [[List](#user-content-updatetype)].|
|lastInfo    |String32767 |O|-|Insert to here host group information of last. A result of [getLastInfo](#user-content-getlastinfomethod) is constructed from this information.|

***hostGroups object***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|groupId  |String255|M|-|Id of host group.|
|groupName|String255|M|-|Host group name corresponds the above group Id.|

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

***Result(result)***

Caller receives result as a result object value whether sent data has been updated. Refer to the [[Values](#user-content-putresult)].

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### putHostGroupMembership(method)

Send all host group membership information to Hatohol server to use "ALL" option when complete connection to Hatohol server and changed hosts information that saved in internal.

When using "UPDATE" option, send difference to use lastInfo that got using [getLastInfo](#user-content-getlastinfomethod) or saved in internal to Hatohol server.

***Request(params)***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|hostGroupMembership|object array|M|-|This is an object which contains host group membership information. In more detail, please refer the following table.|
|updateType|string    |M|-|Select sending option that match the situation from [[List](#user-content-updatetype)].|
|lastInfo    |String32767 |O|-|Put latest host group membership information. A result of [getLastInfo](#user-content-getlastinfomethod) is constructed from this information.|

***hostGroupMembership object***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|hostId  |String255    |M|-|Id of host.|
|groupIds|String255 array|M|-|Id of host group.|

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

***Result(result)***

Caller receives result as a result object value whether sent data has been updated. Refer to the [[Values](#user-content-putresult)].

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### putTriggers(method)

Send all triggers information to Hatohol server to use "ALL" option when complete connection to Hatohol server or reseive [fetchTriggers](#user-content-fetchtriggersmethod) procedure from Hatohol server.

When using "UPDATE" option, send difference to use lastInfo that got using [getLastInfo](#user-content-getlastinfomethod) or saved in internal to Hatohol server.

***Request(params)***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|triggers    |object array|M|-|This is an object array that stores triggers information. In more detail, please refer the following table.|
|updateType|string    |M|-|Select sending option that match the situation from [[List](#user-content-updatetype)].|
|lastInfo    |String32767|O|-|Put to here trigger information of last. A result of [getLastInfo](#user-content-getlastinfomethod) is constructed from this information.|
|fetchId|String255|O|-|This is an ID that indicates whether response for request from Hatohol server. Insert fetchId value to this object, when receiving fetchTriggers request.

***triggers object***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|triggerId     |String255  |M|-|Id of the trigger. When send trigger of HAP itself, must insert "_SELF_" to trigger Id and host Id.|
|status        |string     |M|-|Status of trigger [[List](#user-content-triggerstatus)].|
|severity      |string     |M|-|Severity of trigger [[List](#user-content-triggerseverity)].|
|lastChangeTime|TimeStamp  |M|-|Time of last updated trigger.|
|hostId        |String255  |M|-|An Id of host which belongs to a trigger.|
|hostName      |String255  |M|-|A host name of host which belongs to a trigger.|
|brief         |String255  |M|-|Overview of trigger.|
|extendedInfo  |String32767|M|-|An extra information which is used for the purpose other than the above brief. It is used to show data on WebUI mainly.|

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

***Result(result)***

Caller receives result as a result object value whether sent data has been updated. Refer to the [[Values](#user-content-putresult)].

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### putEvents(method)

If users want to send over than 1000 event with putEvents procedure, users should be separate request which contains less than 1000 events and send them.

Users can send event which duplicates event id with this procedure.

This procedure has two behavior. One is sending spontaneously, the other is sending as a response for fetchEvent procedure.

When sending difference from before events to use lastInfo that got using [getLastInfo](#user-content-getlastinfomethod) or saved in internal to Hatohol server. When HAP connect first time, lastInfo is nothing, so HAP producer can select to send all existing event or do nothing.

If ahead event from to designate event is nothing, send events object as empty string. 

***Request(params)***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|events     |object array|M|-|This is an object array that stores events information. In more detail, please refer the following table.|
|lastInfo    |String32767|O|-|Insert to here event information of last. A result of [getLastInfo](#user-content-getlastinfomethod) is constructed from this information.|
|mayMoreFlag|Boolean   |O|-|Only use in case of response to fetchEvents with fetchId. If plugin has remaining events to be transmitted, must change value to true. In case of that, must send event at least one.|
|fetchId|String255|O|-|This is an ID that indicates whether response for request from Hatohol server. Insert fetchId value to this object, when receiving fetchEvents request.|

***events object***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|eventId     |String255  |M|-|Id of event.|
|time        |TimeStamp  |M|-|Time of happened event.|
|type        |string     |M|-|Type of event.[[eventType](#user-content-eventtype)]|
|triggerId   |String255  |O|-|Trigger id of fired this event. This field is not mandatory because event does not associate with trigger in some cases.|
|status      |string     |O|-|Status of trigger. [[triggerStatus](#user-content-triggerstatus)]|
|severity    |string     |O|-|Severity of trigger. [[triggerSeverity](#user-content-triggerseverity)]|
|hostId      |String255  |O|-|Host id of happened event.|
|hostName    |String255  |O|-|Host name of happened event.|
|brief       |String255  |M|-|Explanation of event. It is shown by on the WebUI.|
|extendedInfo|String32767|O|-|This field can store an additional arbitrary information.|

```json
{
  "id": 1,
  "params": {
    "fetchId": "1",
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

***Result(result)***

Caller receives result as a result object value whether sent data has been updated. Refer to the [[Values](#user-content-putresult)].

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### putHostParents(method)

When HAP completes to connect to Hatohol server,  send all host parent relations to use "ALL" option to Hatohol server.

When using "UPDATE" option, send difference of host parent relations that based on lastInfo that got from [getLastInfo](#user-content-getlastinfomethod) or saved in internal to Hatohol server. 

***Request(params)***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|hostParents  |object array|M|-|This is an object array that stores host parent relations information. In more detail, please refer the following table.|
|updateType|string    |M|-|Select sending option that match the situation from [[List](#user-content-updatetype)].|
|lastInfo    |String32767|O|-|Insert to here host parent relation information of last. A result of [getLastInfo](#user-content-getlastinfomethod) is constructed from this information.|

***hostParents object***

When removing host parent relation from Hatohol server, please send hostParent object that contains empty string as parentHostId.

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|childHostId |String255|M|-|Child host id.|
|parentHostId|String255|M|-|Parent host id.|

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

***Result(result)***

Caller receives result as a result object value whether sent data has been updated. Refer to the [[Values](#user-content-putresult)].

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### finishPut(method)

HAP can divides put procedure that is responce of fetch procedure.finishPut notifies to finish sending all messages to Hatohol Server.

***Request(params)***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|fetchId|String255|O|-|This is an ID that indicates whether response for request from Hatohol server.|

```json
{
  "id": 1,
  "params": {
    "fetchId": 10,
  },
  "method": "finishPut",
  "jsonrpc": "2.0"
}
```

***Result(result)***

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### putArmInfo(method)

Standard behavior is to send after each to send host, trigger and event information to Hatohol server, but users can send at the arbitrary point of the time. The minimum interval MUST be 1 second. The maximum interval SHOULD be twice of polling interval seconds which is obtained by [getMonitoringServerInfo](#user-content-getmonitoringserverinfomethod) and [updateMonitoringServerInfo](#user-content-updatemonitoringserverinfomethod).

***Request(params)***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|lastStatus         |string   |M|-|Latest polling result. [[armInfoStatus](#user-content-arminfostatus)]|
|failureReason      |String255|M|-|Reason of fail to get information. |
|lastSuccessTime    |TimeStamp|M|-|Time of latest success to get information. If have not been successful once, set empty string.|
|lastFailureTime    |TimeStamp|M|-|Time of latest failure to get information. If have not been unsuccessful once, set empty string.|
|numSuccess         |Number   |M|-|The number of success to get information from start up HAP.|
|numFailure         |Number   |M|-|The number of failure to get information from start up HAP.|

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

***Result(result)***

Caller receives result as a result object value whether sent data has been updated. Refer to the [[Values](#user-content-putresult)].

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### fetchItems(method)

This procedure requests to obtain item to HAP. HAP MUST return success or failure to tell accept or not when receiving a request. After that, Send to matched items using a putItems procedure. At that time, need to pass fetchId and hostIds of fetchItems procedure params to putItems procedure.

***Request(params)***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|hostIds|String255 array|O|-|Narrow down host(s) to obtain triggers. If not specifying hostId(s), all trigger should be subject to this procedure.|
|fetchId|String255    |M|-|This field is used in putItems procedures. It needs to identify the putItems correspondence to the fetchItems by Hatohol server.|

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

***Result(result)***

Return a result as a result object value whether request accept to Hatohol server. Refer to the [[Values](#user-content-putresult)].

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### fetchHistory(method)

This procedure is called when Hatohol server makes a request to HAP for obtaining history. HAP must return success or failure of to accept a that request. After that, Send to matched history using a putHistory procedure. At that time, it needs to pass fetchId which is contained in fetchHistory procedure paramemeters to putHistory procedure.

It assumes to obtain history which satisfies conditions. The condition is described in detail as follows:
beginTime and endTime contains in params object. History timestamps satisfies grater than beginTime and less than endTime.

***Request(params)***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|hostId    |String255|M|-|Host Id that the item belongs.|
|itemId    |String255|M|-|Item Id of history.|
|beginTime |TimeStamp|M|-|Starting point of history.|
|endTime   |TimeStamp|M|-|End point of history.|
|fetchId|String255    |M|-|This field is used in putHistory procedures. It needs to identify the putHistory correspondence to the fetchHistory by Hatohol server.|

```json
{
  "id": 1,
  "params": {
    "fetchId": 1,
    "beginTime": "20150323151300",
    "valueType": "INTERGER",
    "itemId": 1,
    "hostId": "1"
  },
  "method": "fetchHistory",
  "jsonrpc": "2.0"
}
```

***Result(result)***

Return a result as a result object value whether request accept to Hatohol server. Refer to the [[Values](#user-content-putresult)].

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### fetchTriggers(method)

Hatohol server calls this procedure to obtain triggers to HAP. HAP MUST return success or failure to tell accept or not when receiving a request. After that, HAP sends to triggers which belongs to host(s) specified with "ALL" option. And then, it needs to pass fetchId which is contained in fetchTriggers procedure paramemeters to putTriggers procedure.

***Request(params)***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|hostIds|String255 array|O|-|Get triggers of to designate a hosts. If does not designate hostIds, all trigger are targeted.|
|fetchId|String255    |M|-|This field is used in putTriggers procedures. It needs to identify the putTriggers correspondence to the fetchTriggers by Hatohol server:.|


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

***Result(result)***

Return a result as a result object value whether request accept to Hatohol server. Refer to the [[Values](#user-content-putresult)].

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### fetchEvents(method)

This procedure request to HAP to obtain designate number of events which are sorted by ascending or descending order. HAP must return to Hatohol server to tell request result whether success or failure. After that, HAP sends events which satisfies condition to Hatohol server with putEvents procedure. And then, it needs to pass fetchId of fetchEvents procedure params to putEvents procedure.

The maximum value of "count" field in this procedure is 1000. If Hatohol server want to request over 1000 events, split the request into two or more to make count field value less than 1000.

***Request(params)***

|Object name|Type |M/O|Default value|Description|
|:-----------------|:--|:-:|:----------:|:---|
|lastInfo |String32767|M|-|Basis of ivent information.|
|count    |Number   |M|-|Event count of getting.|
|direction|String255|M|-|Select "ASC"(newer event than designate ID) or "DESC"(older event that designate ID).|
|fetchId  |String255|M|-|This field is used in putEvents procedures. It needs to identify the putEvents correspondence to the fetchEvents by Hatohol server .|

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

***Result(result)***

Return a result as a result object value whether request accept to Hatohol server. Refer to the [[Values](#user-content-putresult)].

```json
{
  "id": 1,
  "result": "SUCCESS",
  "jsonrpc": "2.0"
}
```

### updateMonitoringServerInfo(notification)

This procedure is sent from Hatohol server when monitoring server information is updated. At that time, HAP should update each monitoring server information by restarting itself. This is standard behavior.

***params***

params content is identical to the result object of [getMonitoringServerInfo](#user-content-getmonitoringserverinfomethod).

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

## Tables

### armInfoStatus

|Status|Description|
|:---------|:---|
|"INIT"   |Initial status. This status represents that not communication yet.|
|"OK"     |Succeed connection.|
|"NG"     |Fail connection.|

### ServerType

If you create a HAP, it needs to define new server type which has no existence UUID as below table.

|Name|UUID|
|:---|:---|
|Zabbix    |8e632c14-d1f7-11e4-8350-d43d7e3146fb|
|Nagios    |902d955c-d1f7-11e4-80f9-d43d7e3146fb|
|Ceilometer|aa25a332-d1f7-11e4-80b4-d43d7e3146fb|

### triggerSeverity

|Severity|Description|
|:---|:--|
|"UNKNOWN"  |Unknown|
|"INFO"     |Information|
|"WARNING"  |Warning|
|"ERROR"    |Error|
|"CRITICAL" |Critical|
|"EMERGENCY"|Emergency|

### triggerStatus

|Status|Description|
|:---------|:---|
|"OK""    |Succeed connection.|
|"NG"     |Fail connection.|
|"UNKNOWN"|Status is unknown|

### updateType

|Type|Description|
|:---------|:---|
|"ALL"    |Send all data. Remove old data from hatohol server and regist all sent data.|
|"UPDATED"|Send only updated data. After that if data ID(s) already exist in Hatohol server, they will be overwritten. Or, if they do not exist in Hatohol Server, they are newly registered in Hatohol server.|

### eventType

|Type|Description|
|:---|:---------:|
|"GOOD"        |Normal|
|"BAD"         |Abnormal|
|"UNKNOWN"     |Unknown|
|"NOTIFICATION"|Notification|

### putResult

The following table values represent whether success or failure when calling put procedure series. If it fails to update database, recalling procedure is standard behavior.

|Status|Description|
|:---------|:---|
|"SUCCESS"|Update is succeed.|
|"FAILURE" |Update is fail.|

If put series of procedures' parameters are wrong or invalid and so on, it needs to return error which is defined in the JSON RPC 2.0 to HAP as following example:

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

The following table values represent whether success or failure when calling fetch series of procedures. If a request fails, resending request to plugin is standard behavior.

|Status|Description|
|:---------|:---|
|"SUCCESS"|Succeed to accept a request.|
|"ABBREV" |Abbreviate to accept a request because request interval a little.| 
|"FAILURE"|Fail to accept a request.|

If fetch series of procedures' parameters are wrong or invalid and so on, it needs to return error which is defined in the JSON RPC 2.0 to HAP as following example:
 
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
## Revision history
-->

## Getting help 
If you find a bug in this documentation or propose an improvement, please contact to us from Hatohol community mailing list. [hatohol-users@lists.sourceforge.net]

When this English specifications differ from Japanese ones, latter specifications have priority.

## Copyright
Copyright (C)2015 Project Hatohol
