# **FIX API**

Introduction

```
Exchange API is an interface provided by Coinsuper for external developers.
```

------

Update log

| Version   | Modify date | Comment       | Author |
| --------- | ----------- | ------------- | ------ |
| v1.0.0(α) | 20181226    | First edition | liyong |

------------- ----------------- --------------- ------------
## 一、**General Information**

#### **API Access Address**

-   API access address: apifix.coinsuper.com
-   API access portal: 1443
-   API access protocol: TCP(SSL)

#### **API Specifications**

-   Access using the SSL socket keep-alive connection.

-   All API requests are made as FIX (4.4) requests.

-   API request data should meet the FIX protocol format.

-   API return values should meet the FIX protocol format.

-   When the server-side receives user requests, responses will follow later. The specific response formats are referred to the following parameter list.

-   Log-on is required for API calls. For log-on methods and parameters, please refer to the following parameter list.

##### **POST Body Format**

Parameter Details

```
1. Parameter Descriptions:
  1.1 Request Parameters:
  Client-side fixed parameters:
  BeginString=FIX.4.4
  TargetCompID=COINSUPER
  HeartBtInt=30
  1.2 Response Parameters:
	For specific responses, please refer to the following API list. Please refer to the provided table to find out what exception has occurred.
	
2. Quotation accuracy:
  2.1 For crypto-crypto transactions, the price is accurate to 8 decimal places and the quantity is accurate to 4 decimal places.
  2.2 For crypto-fiat transactions, the price is accurate to 2 decimal places and the quantity is accurate to 4 decimal places.
  2.3 The API servers will always use this degree of accuracy, regardless of decimal places of request inputs in the "data" parameters.
  
3. Signature:
  All requests must contain a valid signature. The signature generation method is detailed below.

4. Other market data may be acquired through the REST API or WebSocket API.

5. Logon messages shall be delivered before a request is sent.
```

##### **Logon Signature Generation Method**

```
Our API requires a signature to be generated, to verify that information has not been tampered with or falsified by bad actors. Therefore, the generation method of sign strings shall be defined.

a. During logon requests, sort the parameters, such as SendingTime, MsgType, MsgSeqNum, SenderCompID, TargetCompID, $secretKey, according to the parameter name (small to large), connect them with "," and then combine the MD5 encrypted signature to generate sign.

b. During logon requests, the required parameters include SendingTime, MsgType, MsgSeqNum, SenderCompID and TargetCompID.

c. During logon requests, please do not send $secretKey to secure the accounts.

d. During logon requests, "sign" shall be added as RawData to the Logon message, with words in lower case.

Note:
SenderCompID=$accessKey；
$accessKey and $secretKey are acquired by API access.
```

[Get API Access](https://www.coinsuper.com/setting/apiCenter)

##### **Signature Generation Sample**

```
Let us suppose there is a request with these parameters, among which,
$accesskey : zhangsan，$secretkey : zhangsan，TargetCompID : SERVERTARGET

Logon Request Parameters:
"MsgSeqNum" -> "1"
"MsgType" -> "A"
"SenderCompID" -> "zhangsan"
"SendingTime" -> "20181228-13:08:08.091"
"TargetCompID" -> "SERVERTARGET"

i: The sorted pre-encryption signature string then is:
1,A,zhangsan,20181228-13:26:54.497,SERVERTARGET,zhangsan

ii: Finally, we encrypt the raw signature string and assign it to signValue. This will be the "sign" for the actual request body.
signValue = md5(string);

Complete Request Body:
8=FIX.4.49=11735=A34=149=zhangsan52=20181228-13:26:54.49756=SERVERTARGET95=3296=74c544ec967aae34fe84a30bae59520798=0108=3010=188
```

#### FIX is requesting a standard message header.

Hint: every request of interface must convey the correspondent parameters of the message. The definitions of interfaces and exemples of requests are listed as follows that may help to understand specific operation methods.

| Field Name   | Field code | Type of Insertion | Discription                              |
| ------------ | ---------- | ----------------- | ---------------------------------------- |
| BeginString  | 8          | Compulsory        | The version of FIX Protocol  (Fixed value: FIX. 4.4) |
| BodyLength   | 9          | Compulsory        | As to the length of any message,  the number of bytes should be stably controlled at the second field of the  whole message (unencrypted). |
| MsgType      | 35         | Compulsory        | The type of request message.             |
| MsgSeqNum    | 34         | Compulsory        | The series number of the  integral message (int message). |
| SenderCompID | 49         | Compulsory        | $accesskey, acquired via API  activation. |
| SendingTime  | 52         | Compulsory        | Request on sending the time (UTC  time). |
| TargetCompID | 56         | Compulsory        | The signal of server, please  read the [login signature discription] above for references. |

#### **General Exceptions**

| **Exception**                | Description                              |
| ---------------------------- | ---------------------------------------- |
| user not exist!              | Wrong TargetCompID                       |
| has no authentication        | There is no corresponding API access     |
| request too frequently       | Requests exceed the threshold            |
| SendingTime accuracy problem | Client UTC time does not match server UTC time (Please compare it to UTC) |
| system internal error        | system internal error                    |

 Note: Exceptions are usually responded with "Reject". All APIs shall contain more detailed information.
------------------------ ---------------------------------------


## **二、 API Endpoint Details**

### API Endpoint Definition

#### **1. Session Type**

##### **1.1 Logon**

Description:

Logon Request (Keep-alive connection sessions are created upon successful logons, and other requests depend on successful logons)

Request MsgType:

```
Logon
```

Logon

API Request Parameters:

| **Parameter** | **Field Code** | **Mandatory** | Description                              |
| ------------- | -------------- | ------------- | ---------------------------------------- |
| SendingTime   | 52             | Yes           | Request delivery time (UTC time)         |
| MsgType       | 35             | Yes           | Message type                             |
| MsgSeqNum     | 34             | Yes           | Message sequence number                  |
| SenderCompID  | 49             | Yes           | API user accessKey                       |
| TargetCompID  | 56             | Yes           | For fixed parameters, please refer to values as described in the abovementioned \[parameter details\]. |
| RawData       | 96             | Yes           | Logon parameter signature (for valid signatures, please refer to the abovementioned \[Logon Signature Generation Method\] |

Response MsgType：

```
Logon
```

Response Parameters:

| **Parameter** | Field Code | **Description**                 |
| ------------- | ---------- | ------------------------------- |
| ClOrdID       | 11         | User's Account Number           |
| OrderID       | 37         | User's e-mail address           |
| ExecType      | 150        | System timestamp (Milliseconds) |
| OrdStatus     | 39         | Return results                  |
| TransactTime  | 60         | Total balance                   |

Request Sample:

```
8=FIX.4.49=11435=A34=249=zhangsan52=20190102-03:41:14.32956=COINSUPER95=3296=6cc719376923d980cbb5c882191d4e2898=0108=3010=058
```

Response Sample:

```
8=FIX.4.49=7235=A34=449=COINSUPER52=20190102-03:41:14.46156=zhangsan98=0108=3010=011
```

------

##### **1.2 Logout**

Description:

Upon initiation of Logout request, the keep-alive connection session will be closed.

Request MsgType:

```
Logout
```

API Request Parameters:

None

Response MsgType:

```
Logout
```

Response Parameters:

None

Request Sample:

```
8=FIX.4.49=11435=534=249=zhangsan52=20190102-03:41:14.32956=COINSUPER95=3296=6cc719376923d980cbb5c882191d4e2898=0108=3010=058
```

Response Sample:

```
8=FIX.4.49=7235=534=449=COINSUPER52=20190102-03:41:14.46156=zhangsan98=0108=3010=011
```

------

##### 1.3 Heartbeat

Description:

Heartbeat requests are delivered for a fixed time, which aims to maintain keep-alive connection sessions.

Request MsgType:

```
Heartbeat
```

API Request Parameters:

None

Response MsgType:

```
Heartbeat
```

Response Parameters:

None

Request Sample:

```
8=FIX.4.49=6035=034=349=zhangsan52=20190102-07:31:01.57256=COINSUPER10=223
```

Response Sample:

```
8=FIX.4.49=6235=034=46349=COINSUPER52=20190102-07:31:01.69056=zhangsan10=076
```

------

#### 2. Transaction Type

##### **2.1 Order Creation**

Description:

Transaction Order Creation Request

Request MsgType:

```
NewOrderSingle
```

API Request Parameters:

| **Parameter** | **Field Code** | **Mandatory** | **Description**                          |
| ------------- | -------------- | ------------- | ---------------------------------------- |
| ClOrdID       | 11             | Yes           | Customized order ID from the client-side (cannot repeat) |
| Symbol        | 55             | Yes           | Trading pairs                            |
| Price         | 44             | Yes           | Trading price limit (which is exclusive for price limit orders, and the market price should be 0) |
| Side          | 54             | Yes           | Purchase and sales type (Buy (1)= place a buy order, SELL (2)=place a sell order) |
| OrdType       | 40             | Yes           | Order type (MARKET (1)=market price, and LIMIT (2) = limit price) |
| OrderQty      | 38             | Yes           | Quantity of target tokens (used for purchases and sales at limit prices or sales at market prices. Please input "0" for purchases at market prices) |
| CashOrderQty  | 152            | Yes           | Quantity of valued tokens (used for purchases at market prices. Please input "0" in case of purchases and sales at limit prices or sales at market prices) |
| TransactTime  | 60             | Yes           | Request time (UTC time)                  |

```
Additional Parameter Descriptions:

For BTC/USD trading pairs, see the sample below regarding OrderQty and Amount:

Buy 0.1 BTC at the limit price of USD 6300:
Symbol=BTC/USD, Side=1, OrdType=2, Price=6300, OrderQty=0.1, CashOrderQty=0;

Sell 0.2 BTC at the limit price of USD 6301:
Symbol=BTC/USD, Side=2, OrdType=2, Price=6300, OrderQty=0.2, CashOrderQty=0;

Buy BTC at the market price of USD 500:
Symbol=BTC/USD, Side=1, OrdType=1, Price=0, OrderQty=0, CashOrderQty=500;

Sell 0.5 BTC at the market price:
Symbol=BTC/USD, Side=2, OrdType=1, Price=0, OrderQty=0.5, CashOrderQty=0;
```

Response MsgType:

```
ExecutionReport
```

Response Parameters:

| **Parameter** | Field Code | **Description**                          |
| ------------- | ---------- | ---------------------------------------- |
| ClOrdID       | 11         | Customized order ID from the client-side |
| OrderID       | 37         | Order ID generated from the server-side  |
| ExecType      | 150        | Execution results (fixed as NEW)         |
| OrdStatus     | 39         | Order status (fixed as NEW)              |
| TransactTime  | 60         | Delivery time of response messages (UTC time) |

Request Sample：  

```
8=FIX.4.49=16735=D34=3249=zhangsan52=20190104-10:08:43.31456=COINSUPER11=ord000138=0.340=244=450054=255=BTC/USD60=20190104-18:08:43.308152=010=091
```

 Response Sample: 

```
8=FIX.4.49=22335=834=18349=COINSUPER52=20190104-10:08:42.34256=zhangsan6=014=017=a0cbfd23455e4b6faaacbe5fb36caf9920=037=162172399493790924939=054=255=BTC/USD60=20190104-18:08:42.341150=0151=0.310=036
```

| **Exception**                            | **Description**                          |
| ---------------------------------------- | ---------------------------------------- |
| symbol not trading                       | Trading pairs are not availablefortrading |
| order amount or quantity less than min setting | The trading quantity/amount is lower than the minimum requirement |
| tprice out of range                      | The price exceeds the range              |
| action not support                       | It does not support such transaction type |
| order type not support                   | It does not support such order type      |
| user account forbidden                   | The account is forbidden from trading    |
| balance not enough                       | The balance is insufficient              |

------

##### 2.2 Order Cancellation

Description:

Transaction Order Cancellation Requests

Request MsgType:

```
OrderCancelRequest
```

API Request Parameters:

| **Parameter** | Field Code | **Mandatory** | **Description**                         |
| ------------- | ---------- | ------------- | --------------------------------------- |
| OrderID       | 37         | Yes           | Order ID generated from the server-side |

Response MsgType:

```
ExecutionReport
```

Response Parameters:

| Parameter    | Field Code | **Description**                          |
| ------------ | ---------- | ---------------------------------------- |
| OrderID      | 37         | Order ID generated from the server-side  |
| ExecType     | 150        | Execution results (order cancellation in progress: PENDING\_CANCEL) |
| OrdStatus    | 39         | Order status (order cancellation in progress: PENDING\_CANCEL) |
| TransactTime | 60         | Delivery time of response messages (UTC time) |

Request Sample:  

```
8=FIX.4.49=11235=F34=6549=zhangsan52=20190104-09:40:25.49556=COINSUPER37=162172076101837209710=112
```

 Responce Sample: 

```
8=FIX.4.49=15435=834=12749=COINSUPER52=20190104-09:40:24.38956=zhangsan20=137=162172076101837209739=460=20190104-17:40:24.389150=410=041
```

Upon failures due to various reasons, values shall be obtained from the response parameters:

| **Exception**      | Description                      |
| ------------------ | -------------------------------- |
| order no not exist | There exists no unfinished order |
| order has canceled | The order has been canceled      |
| order has execute  | The order has been completed     |

-------------------- -----------------------------------
####3. Query Type

##### **3.1 Queries into Unfinished Orders**

Description:

Queries are made into partially executed and in-progress orders

Request MsgType:

```
OrderStatusRequest
```

API Request Parameters:

| Parameter | Field Code | Mandatory | Description                              |
| --------- | ---------- | --------- | ---------------------------------------- |
| OrderID   | 37         | Yes       | Order ID generated from the server-side (\* indicates that queries are made into the latest 20 order entries) |

Response MsgType:

```
ExecutionReport
```

Response Parameters:

| **Parameter** | Field Code | **Description**                          |
| ------------- | ---------- | ---------------------------------------- |
| OrderID       | 37         | Order ID generated from the server-side  |
| Side          | 54         | Purchase and sales type (Buy (1)= place a buy order, SELL (2)=place a sell order) |
| ExecType      | 150        | Execution results (ORDER\_STATUS)        |
| OrdStatus     | 39         | Order status (PENDING\_NEW/PARTIALLY\_FILLED) |
| AvgPx         | 6          | Average transaction price per order      |
| CumQty        | 14         | Quantity of completed transactions       |
| LeavesQty     | 151        | Quantity of remaining unfinished transactions (CumQty+LeavesQty= total order quantity) |
| Symbol        | 55         | Trading pairs                            |
| TransactTime  | 60         | Delivery time of response messages (UTC time) |

Request Sample:

```
8=FIX.4.49=11235=H34=1849=zhangsan52=20190104-10:01:52.86256=COINSUPER37=162172183852353945710=111
```

 Response Sample: 

```
8=FIX.4.49=22335=834=16949=COINSUPER52=20190104-10:01:51.70456=dba8ef1f-6e3f-43ed-ae67-0668c3e933636=014=017=37e01f58367948fdaedf8dace2ea62ff20=337=162172183852353945739=A54=255=BTC/USD60=20190104-18:01:51.704150=I151=0.210=199
```

Upon failures due to various reasons, values shall be obtained from the response parameters:

| **Exception**      | **Description**                  |
| ------------------ | -------------------------------- |
| order no not exist | There exists no unfinished order |
