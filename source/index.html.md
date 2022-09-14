--- 

title: Zero Hash Matching Engine FIX API Documentation

language_tabs: 
   - fix: FIX

toc_footers: 
   - <a href="https://zerohash.com/">Zero Hash</a> 
   

# includes: 
#    - errors 

search: true 

--- 

# INTRODUCTION 

## Overview

Welcome to Zero Hash API documentation. These documents outline trading platform functionality, market details, and APIs.

APIs are separated into two categories: Market data feed and trading. Feed APIs provide market data and are public. Trading APIs require authentication and provide access to placing orders and other account information. 

**Version:** 1.0.0 

## Rate Limits

The Zero Hash API is rate limited to prevent abuse that would degrade our ability to maintain consistent API performance for all users.

### Financial Information Exchange API

The FIX API throttles the number of incoming messages to 10 commands per second by default. This can be increased as needed.

## Changelog

Recent changes and additions to Zero Hash API.

### 2021-01-30

Initial Version

### 2022-09-01

CERT and Production 

# Supported Orders

The table below details what Order Types and associated Time In Force are supported over the FIX API.

| Order Type  | Time In Force    |
| ----------- | ---------------- |
| Limit       | DAY <br/> GTC <br/> IOC <br/> GTD |


# <br/>

# FIX API

The information in Zero Hash FIX API specification describes the adaptation of the standard FIX 4.2 for vendors and subscribers to communicate with the Zero Hash quotation and execution platform. FIX 4.2 tags, as described in detail on the Financial Information Exchange Protocol Committee website, www.fixprotocol.org as well as custom tags are used extensively in this document and the reader should familiarize themselves with the latest updates to this release. If an application message in Financial Information Exchange Protocol version 4.2, or previous FIX versions, is not included in this document, the message is ignored.


# Connectivity
To connect to Zero Hash over the FIX API clients must establish a secure connection to the FIX gateway. you can Connect to the FIX gateway with secure tcp+ssl connection without whitelisting your ip. For a FIX implementation that does not support establishing a TCP SSL connection natively, you will need to setup a local proxy such as stunnel to establish a secure connection to the FIX gateway. if these two ways not working with you then you to provide us connectivity details such as the IP Address and Port Number which will be assigned by the Zero Hash trade operations team. To request these details and to Whitelist clients' IP addresses, clients can contact support@zerohash.com or raise a request via the support portal through the www.zerohash.com website.

<i>Message Sequence Numbers are reset daily at 01:00 UTC.</i>

For testing use;

IP Address: `FIRMID.oe.cert.zerohash.com`<br />
Port: `13010`<br/>
SenderComp: `FIRM-SENDER-COMP-1`


For access to Market Data over FIX use;

IP Address: `FIRMID.md.cert.zerohash.com`<br />
Port: `13010`<br/>
SenderComp: `FIRM-SENDER-COMP-2`



# FIX Order Placement

The trading platform will respond to any new order messages and trade with execution reports. Example execution report responses are available below.

<br/>

This section outlines the FIX messages, how they are supported, and to what extent the business data is translated within the FIX Gateway.

### Message Header

<br/>

| Tag  | Field Name              | Required  | Note |
| ---- | ----------------------- | --------  | ---- |
| 8    | BeginString             | Yes       | FIX.5.0 |
| 9    | BodyLength              | Yes       | (Always unencrypted, must be second field in message)      |
| 35   | MsgType                 | Yes       |  (Always unencrypted, must be third field in message)    |
| 49   | SenderCompID            | Yes       | Provided by Zero Hash |
| 56   | TargetCompID            | Yes       | Provided by Zero Hash |
| 34   | MsgSeqNum               | Yes       | (Can be embedded within encrypted data section.)     |
| 52   | SendingTime             | Yes       |  Required    |


### Message Trailer

| Tag  | Field Name              | Required  | Note |
| ---- | ----------------------- | --------  | ---- |
| 10   | CheckSum                | Yes       | (Always unencrypted, always last field in message)   |


## Login (A)

```fix
Example Message

Client
8    =  FIXT.1.1
9    =  80
35   =  A
34   =  1
49   =  ZEROTEST
52   =  20220911-00:01:47.190
56   =  ZERO
98   =  0
108  =  60
141  =  Y
1137 =  9
10   =  118

Gateway
8    =  FIXT.1.1
9    =  86
35   =  A
34   =  1
49   =  ZERO
52   =  20220911-00:01:47.237480372
56   =  ZEROTEST
98   =  0
108  =  60
141  =  Y
1137 =  9
10   =  182
```

The logon message identifies and authenticates the user or member and establishes a connection to the FIX Gateway. 
The FIX gateway accepts Logon messages as per the FIX specification. Further, the FIX gateway supports the logon sequence required for session authentication. 
After a successful logon as described in the specification the FIX gateway will: 
Initiate retransmission processing via a resend request if the Logon sequence number is greater than the value expected
Initiate logout processing via a Logout message with an appropriate error message, then waits for a confirming Logout before disconnecting if the Logon sequence number is less than expected. If the confirming Logout has not been received within a short period of time the session will be disconnected.
Handle retransmission requests.
Initiate a Logon using the SenderCompID in the message header.
Will forwarded to the FIX client messages that are waiting in the outbound queue
Begin regular message communication.
<br/>

| Tag  | Field Name              | Required  | Note |
| ---- | ----------------------- | --------  | ---- |
|      | Standard Header         | Yes       | MsgType = A  |
| 98   | EncryptMethod           | Yes       | 0 = NONE_OTHER <br/> 1 = PKCS <br/> 2 = DES <br/> 3 = PKCS_DES <br/> 4 = PGP_DES <br/> 5 = PGP_DES_MD5<br/> 6 = PEM_DES_MD5 |
| 108  | HeartBtInt              | Yes       |  |
| 141  | ResetSeqNumFlag         | No        |  |
| 1137 | DefaultApplVerID        | Yes       |  |
|      | Standard Trailer        | Yes       |  |



## Logout (5)

```fix
Example Message


Client
8  =  FIXT.1.1
9  =  53
35 =  5
34 =  6
49 =  ZEROTEST
52 =  20220910-17:30:00.366
56 =  ZERO
10 =  00
```

The Logout message initiates or confirms the termination of a FIX session. 

The FIX gateway will receive and generate logout messages as required by the FIX Protocol. The gateway follows the prescribed sequence of messages for the proper termination of the session. 
Messages received by the gateway after the client logs out are stored in a log file for transmission to the client once the client logs in again within the same trading day. The messages to be transmitted are dependent on the sequence number reconciliation that occurs on a logon handshake. 

Upon receipt of a Logout message: 
<br/>
1. A confirming logout message will be sent by the gateway to the client; then,
<br/>
2. The session will be disconnected. The FIX gateway will not initiate a logoff except when a severe error has occurred.

<br/>

| Tag | Field Name       | Required | Note                                                                                                                                                                                                    |
| --- | ---------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|     | Standard Header  | Yes      | MsgType = 5                                                                                                                                                                                             |
| 58  | Text             | No       | Free format text string (Note: this field does not have a specified maximum length) If the Logout message has been sent by the the FIX gateway, then this field will contain the text “Session closed”. |
|     | Standard Trailer | Yes      |                                                                                                                                                                                                         |




## Heartbeat (0)

```fix
Example Message

Client
8  =  FIXT.1.1
9  =  57
35 =  0
34 =  841
49 =  ZEROTEST
52 =  20220911-14:09:36.472
56 =  ZERO
10 =  087

Gateway
8  =  FIXT.1.1
9  =  63
35 =  0
34 =  849
49 =  ZERO
52 =  20220911-14:09:48.786045303
56 =  ZEROTEST
10 =  150
```

This section indicates the message intended to monitor the status of the communications link during periods of inactivity. 

The FIX market data accepts and generates Heartbeat messages as per the FIX specification.

• Inbound: Handled as specified 
• Outbound: In response to a test request or timeout. 
• Response: None

The heartbeat message should be sent if agreed upon Heartbeatinterval has elapsed since the last message sent. If any 
proceeding Heartbeatinterval a Heartbeat message need not be sent
<br/>

| Tag | Field Name       | Required | Note        |
| --- | ---------------- | -------- | ----------- |
|     | Standard Header  | Yes      | MsgType = 0 |
|     | Standard Trailer | Yes      |             |





## New Order - Single (D)

```fix
Example Message

Client - BTC/USD Buy Limit GTC Order for OrderQty 0.01 at Price of 19000
8   =   FIXT.1.1
9   =   254
35  =   D
34  =   426
49  =   ZEROTEST
50  =   trader
52  =   20220910-07:10:21.970
56  =   ZERO
1   =   firms/Zero-Hash-Liquidity/accounts/test
11  =   3637983906161824000
18  =   2
21  =   1
22  =   8
38  =   0.01
40  =   2
44  =   19000
48  =   BTC/USD
54  =   1
55  =   BTC/USD
58  =   ZH Testing Trades
59  =   1
60  =   20220913-06:01:21.970
167 =   CS
10  =   007


Gateway - Execution Reports

ER NEW
8   =  FIXT.1.1
9   =  394
35  =  8
34  =  11
49  =  ZERO
52  =  20220913-06:09:10.076887737
56  =  ZEROTEST
57  =  trader
1   =  firms/Zero-Hash-Liquidity/accounts/test
6   =  0.000000000
11  =  3637983906161824000
14  =  0.00000000
17  =  1569568851483602944
22  =  8
31  =  0.00
32  =  0.00000000
37  =  1569568851307696128
38  =  0.01000000
39  =  0
40  =  2
44  =  19000.00
48  =  BTC/USD
54  =  1
55  =  BTC/USD
59  =  0
60  =  20220913-06:09:10.029186377
99  =  0.00
150 =  0
151 =  0.01000000
581 =  14
582 =  4
10  =  130

ER FILL
8   =   FIXT.1.1
9   =   408
35  =   8
34  =   13
49  =   ZERO
52  =   20220913-06:09:10.079060678
56  =   ZEROTEST
57  =   trader
1   =   firms/Zero-Hash-Liquidity/accounts/test
6   =   22671.612500000
11  =   3637983906161824000
14  =   0.01000000
17  =   1569568851483602949
22  =   8
31  =   18740.25
32  =   0.01000000
37  =   1569568851307696128
38  =   0.01000000
39  =   2
40  =   2
44  =   19000.00
48  =   BTC/USD
54  =   1
55  =   BTC/USD
59  =   0
60  =   20220913-06:09:10.029186377
99  =   0.00
150 =   F
151 =   0.00000000
581 =   14
582 =   4
828 =   0
10  =   103
```

This message is used to submit an order to the trading system for processing. The trading platform will respond with execution reports.

| Tag | Field Name       | Required | Note                                                                                                                                                                 |
| --- | ---------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|     | Standard Header  | Yes      |                                                                                                                                                                      |
| 11  | ClOrdId          | Yes      | Unique identifier of the order. Must be unique for each session, max 32 chars.                                                                                       |
| 1   | Account          | No       | Optional identifier from customer, will be passed back in Execution Report.                                                                                          |
| 21  | HandlInst        | Yes      | '1' = Automated execution order, private, no Broker intervention '2' = Automated execution order, public, Broker intervention OK  '3' = Manual order, best execution |
| 55  | Symbol           | Yes      | Common, “human understood” representation of the security, bitcoins US Dollar – BTCUSD, etc.                                                                         |
| 48  | Symbol           | Yes      | Symbol                                                                                                                                                               |
| 54  | Side             | Yes      | '1' = Buy '2' = Sell                                                                                                                                                 |
| 38  | OrderQty         | Yes      | Size of the order. E.G. 10                                                                                                                                           |
| 40  | OrdType          | Yes      | '2' = Limit <br/> '3' = Stop <br/> '4' = Stop Limit <br/> 'K' = MARKET_WITH_LEFT_OVER_AS_LIMIT                                                                                                                                             |
| 44  | Price            | No       | Required for limit Ordtypes                                                                                                                                          |
| 59  | TimeInForce      | No       | 0 = DAY <br/> 1 = GOOD_TILL_CANCEL <br/> 3 = IMMEDIATE_OR_CANCEL <br/> 6 = GOOD_TILL_DATE                                                                            |
| 60  | TransactTime     | No       | Time of execution/order creation (UTC Time in datetime format)                                                                                                       |
| 126 | ExpireTime       | No       | Required if TimeInForce = 6 (UTC Time in datetime format - 17:18:02.54)                                                                                              |
|     | Instrument       | Yes      | Component Fields can be found in appendix                                                                                                                            |
|     | Standard Trailer | Yes      |                                                                                                                                                                      |




## Order Cancel Request (F)

```fix
Example Message

Client
8=FIX.4.2^A9=188^A35=F^A34=104^A49=HASFIX02^A52=20210714-12:56:35.070^A56=MIDCDEV1^A11=63637618640333706000^A21=1^A38=1^A40=2^A41=62637618638809898000^A44=125^A54=1^A55=LTCUSD^A59=1^A60=20210714-12:56:35.075^A43229=100^A10=196^A

Gateway
8=FIX.4.2^A9=183^A35=F^A34=724^A49=MIDCDEV1^A52=20210714-12:56:35.080^A56=YOURSENDCOMP^A11=63637618640333706000^A21=2^A38=1^A40=2^A41=62637618638809898000^A44=125^A54=1^A55=LTCUSD^A59=1^A60=20210714-12:56:35.80^A43229=1^A10=094^A

```

| Tag | Field Name       | Required | Note                                                                                         |
| --- | ---------------- | -------- | -------------------------------------------------------------------------------------------- |
|     | Standard Header  | Yes      |                                                                                              |
| 41  | OrigClOrdId      | Yes      | ClOrdID of the previous order                                                                |
| 37  | OrdId            | No       | Unique Identifier of order assigned by the platform.                                         |
| 11  | ClOrdId          | Yes      | Unique identifier of the order. Must be unique for each session, max 32 chars.               |
| 55  | Symbol           | Yes      | Common, “human understood” representation of the security, bitcoins US Dollar – BTCUSD, etc. |
| 54  | Side             | Yes      | '1' = Buy '2' = Sell                                                                         |
| 38  | OrderQty         | Yes      | Size of the order. E.G. 10                                                                   |
| 58  | Text             | No       |                                                                                              |
|     | Standard Trailer | Yes      |                                                                                              |




## Order Cancel/Replace Request (G)

```fix
Example message

Client
8=FIX.4.2^A9=188^A35=F^A34=104^A49=HASFIX02^A52=20210714-12:37:23.030^A56=MIDCDEV1^A11=61637618630042074000^A21=1^A38=1^A40=2^A41=60637618629492018000^A44=47250^A54=1^A55=LTCUSD^A59=1^A60=20210714-12:37:23.35^A43229=100^A10=196^A

Gateway
8=FIX.4.2^A9=183^A35=G^A34=704^A49=MIDCDEV1^A52=20210714-12:37:23.035^A56=YOURSENDCOMP^A11=61637618630042074000^A21=2^A38=1^A40=2^A41=60637618629492018000^A44=47250^A54=1^A55=LTCUSD^A59=1^A60=20210714-12:37:23.35^A43229=1^A10=057^A
```
The Execution Report is automatically generated and sent by the gatway automatically.

| Tag  | Field Name              | Required  | Note |
| ---- | ----------------------- | --------  | ---- |
|      | Standard Header         | Yes       |      |
| 41   | OrigClOrdId             | Yes       | ClOrdID of the previous order |
| 37   | OrdId                   | No        | Unique Identifier of order assigned by the platform. |
| 11   | ClOrdId                 | Yes       | Unique identifier of the order. Must be unique for each session, max 32 chars. |
| 1    | Account                 | No        | Optional identifier from customer, will be passed back in Execution Report. |
| 55   | Symbol                  | Yes       | Common, “human understood” representation of the security, bitcoins US Dollar – BTCUSD, etc. |
| 54   | Side                    | Yes       | '1' = Buy '2' = Sell |
| 38   | OrderQty                | Yes       | Size of the order. E.G. 10  |
| 40   | OrdType                 | Yes       |       |
| 44   | Price                   | No        | Required for limit Ordtypes  |
| 59   | TimeInForce             | No        | '3' = IOC (Immediate-Or-Cancel) |
| 126  | ExpireTime              | No        | Required if TimeInForce = GTT (UTC time in datetime format - 17:18:02.544)
|      | Standard Trailer        | Yes       |       |


## Execution Report (8)

The Execution Report is automatically generated and sent by the gatway automatically.


| Tag   | Field Name             | Required  | Note |
| ----  | ---------------------- | --------  | ---- |
|       | Standard Header        | -         | |
| 37    | OrdId                  | -         | Unique Identifier of order assigned by the platform. |
| 6     | AvgPx                  | -         | |
| 11    | ClOrdId                | -         | Unique identifier of the order copied from customer's request. |
| 41    | OrigClOrdId            | -         | ClOrdID of the previous order |
| 14    | CumQty                 | -         | |
| 17    | ExecID                 | -         | Platform assigned execution ID |
| 20    | ExecTransType          | -         | 0 |
| 32    | LastQty                | -         | |
| 37    | OrderID                | -         | |
| 38    | OrderQty               | -         | Size of the order. E.G. 10|
| 39    | OrdStatus              | -         | 0 = New, 1 = PartiallyFilled, 2 = Filled, 4 = Canceled, 5 = Replaced, 6 = PendingCancel, 8 = Rejected, A = PendingNew, C = Expired, E = PendingReplace |
| 40    | OrdType                | -         | '1' = Market '2' = Limit |
| 44    | Price                  | -         | |
| 54    | Side                   | -         | '1' = Buy  '2' = Sell |
| 55    | Symbol                 | -         | Common, “human understood” representation of the trading pair, bitcoins US Dollar – BTCUSD, etc. |
| 59    | TimeInForce            | -         | |
| 60    | TransactTime           | -         | |
| 126   | ExpireTime             | -         | Time order will expire |
| 150   | ExecType               | -         | |
| 151   | LeavesQty              | -         | |
| 43211 | Execution Venue        | -         | This value can be ignored, will be CROX on FILLS |
| 43212 | Execution Contra Broker| -         | This value can be ignored, will be MIDC on FILLS |
| 43214 | Liquidity Indicator    | -         | ‘P’, provided liquidity ‘T’, took liquidity |
| 43229 | Fraction Base          | -         | The amount to divide qty by to get real value |
| 58    | Text                   | -         | |
|       | Standard Trailer       | -         | |

## Order Cancel Reject (9)
This message is send when a Order Cancel Request or Order Cancel/Replace Request is rejected

| Tag  | Field Name              | Required  | Note |
| ---- | ----------------------- | --------  | ---- |
|      | Standard Header         | -         |      |
| 41   | OrigClOrdId             | -         | ClOrdID of the previous order. |
| 37   | OrdId                   | -         | Unique Identifier of order assigned by the platform. |
| 11   | ClOrdId                 | -         | Unique identifier of the order. Must be unique for each session, max 32 chars. |
| 39   | OrdStatus               | -         | OrdStatus value after this cancel reject is applied. |
| 102  | CxlRejReason            | -         | Code to identify reason for cancel rejection. |
| 58   | Text                    | -         | Optional explanation message. |
| 434  | CxlRejResponseTo        | -         | Unique identifier of the Canceled order. |
| 17   | ExecID                  | -         | Execution id. |
|      | Standard Trailer        | -         |      |

# FIX Market Data

The information in Zero Hash FIX API specification describes the adaptation of the standard FIX 4.2 for vendors and subscribers to communicate with the Zero Hash quotation and execution platform. FIX 4.2 tags, as described in detail on the Financial Information Exchange Protocol Committee website, www.fixprotocol.org as well as custom tags are used extensively in this document and the reader should familiarize themselves with the latest updates to this release. If an application message in Financial Information Exchange Protocol version 4.2, or previous FIX versions, is not included in this document, the message is ignored.

**After you connect to a market data session you will automatically receive msg type X and W for all available trading pairs.**


## Market Data Responses – Top of Book Snapshot (W)

```fix
Example FIX Message

The FIX message would appear as 

8=FIX.4.2^A9=103^A35=W^A34=54^A49=MIDCDEV1^A52=20210713-13:49:37^A56=YOURSENDCOMP^A55=LTCUSD^A262=0^A268=1^A269=2^A270=133.54^A271=0.91^A10=024^A

```

| Tag  | Field Name              | Required  | Note |
| ---- | ----------------------- | --------  | ---- |
|      | Standard Header         |        | MsgType = W |
| 262  | MDReqID              |        | The MDReqID of the MarketDataRequest message. |
| 55   | Symbol             |        | Identifier for the symbol  |
| 268  | NoMDEntries         |        | Number of market data entries in this snapshot. |
| 269  | MDEntryType           |        | Type of market data entry. Valid values: 0 = Bid, 1 = Offer  |
| 270  | MDEntryPx            |        | Price of the market data entry. |
| 271  | MDEntrySize         |        | Qty represented by the Market Data Entry. |
|     | Standard Trailer             |        |  |






## Market Data Responses– Incremental Refresh (X)

```fix
Example FIX Message

The FIX message would appear as 

8=FIX.4.8=FIX.4.2^A9=146^A35=X^A34=77^A49=MIDCDEV1^A52=20210713-14:06:54^A56=YOURSENDCOMP^A262=0^A268=1^A279=0^A269=0^A278=247697979505377327^A55=LTCUSD^A270=120^A271=0.1^A58=62WM7F95YBJTC3^A10=075^A

8=FIX.4.2^A9=146^A35=X^A34=77^A49=MIDCDEV1^A52=20210713-8=FIX.4.2^A9=146^A35=X^A34=78^A49=MIDCDEV1^A52=20210713-14:07:21^A56=YOURSENDCOMP^A262=0^A268=1^A279=0^A269=0^A278=247697979505377328^A55=LTCUSD^A270=121^A271=0.2^A58=62WM7F95YBJUC3^A10=075^A

```
<br/>

| Tag  | Field Name              | Required  | Note |
| ---- | ----------------------- | --------  | ---- |
|      | Standard Header         |        | MsgType = X |
| 262  | MDReqID              |        | The MDReqID of the MarketDataRequest message. |
| 268  | NoMDEntries         |        | Number of market data entries in this snapshot. |
| 279  | MDUpdateAction      |        | The Market Data update action type. It must be the first field in the repeating group. The only valid values are: 0 = New, 2 = Delete |
| 269  | MDEntryType           |        | Type of market data entry. Valid values: 0 = Bid, 1 = Offer, 2=Trade |
| 278  | MDEntryID        |        | Market data identifier |
| 55   | Symbol  |  | Identifier for the symbol |
| 65   | SymbolSfx |  |  |
| 270  | MDEntryPx  |   | Price of the market data entry. |
| 271  | MDEntrySize         |        | Number of shares represented by the Market Data Entry. |
| 58   | Text         |        | Order ID reference |
|      | Standard Trailer             |        |  |



## Market Data Request Reject (Y)

<br/>

| Tag  | Field Name              | Required  | Note |
| ---- | ----------------------- | --------  | ---- |
|      | Standard Header         |        | MsgType = Y |
| 262  | MDReqID              |      | The MDReqID of the MarketDataRequest message.|
| 268  | Text         |        | Reason for the rejection.|
|      | Standard Trailer      |        |  | 

# <br/>