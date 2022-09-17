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

Welcome to Zero Hash Matching Engine FIX API documentation.

The FIX API specification is separated into two categories: Order Entry and Market Data APIs.
The information in the FIX API specification describes the adoptation of the standard FIX 5.0 for vendors and subscribers to communicate with the Zero Hash matching engine. FIX  tags, as described in detail on the Financial Information Exchange Protocol Committee website, www.fixprotocol.org as well as custom tags are used extensively in this document and the reader should familiarize themselves with the latest updates to this release. If an application message in Financial Information Exchange Protocol version 5.0, or previous FIX versions, is not included in this document, the message is ignored.

**Version:** 1.0.0 

## Rate Limits

The Zero Hash API is rate limited to prevent abuse that would degrade our ability to maintain consistent API performance for all users.

The FIX API throttles the number of incoming messages to 10 commands per second by default. This can be increased as needed.

## Changelog

Recent changes and additions to Zero Hash Matching Engine API.

### 2022-09-01

Initial Version

# Supported Orders

The table below details what Order Types and associated Time In Force are supported over the FIX API.

| Order Type  | Time In Force    |
| ----------- | ---------------- |
| Limit       | DAY <br/> GTC <br/> IOC <br/> GTD |

# Connectivity
To connect to the Zero Hash Matching Engine via FIX API, clients must have thier source IP address whitelisted and establish a secure SSL connection to the FIX gateway. If your FIX implementation does not support establishing a SSL connection natively, you will need to setup a local proxy such as stunnel to establish a secure connection to the FIX gateway. To request these details and whitelist your source IP addresses, clients can contact support@zerohash.com.

<i>Message Sequence Numbers are reset on Friday at 17:00 EST.</i>

For testing use;

IP Address: `FIRMID.oe.cert.zerohash.com`<br />
Port: `13010`<br/>
SenderComp: `FIRM-SENDER-COMP-1`

TargetCompID: `ZERO`


For access to Market Data over FIX use;

IP Address: `FIRMID.md.cert.zerohash.com`<br />
Port: `13010`<br/>
SenderComp: `FIRM-SENDER-COMP-2`

TargetCompID: `ZERO`


# <br/>
# SESSION LEVEL MESSAGES

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

# Login (A)

```fix
Example Message

Client
8    =  FIXT.1.1
9    =  80
35   =  A
34   =  1
49   =  YOURSENDERCOMP
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
56   =  YOURSENDERCOMP
98   =  0
108  =  60
141  =  Y
1137 =  9
10   =  182
```

The logon message identifies and authenticates the user or member and establishes a connection to the FIX Gateway. 
The FIX gateway accepts Logon messages as per the FIX specification. Further, the FIX gateway supports the logon sequence required for session authentication. 
After a successful logon as described in the specification the FIX gateway will: 
-Initiate retransmission processing via a resend request if the Logon sequence number is greater than the value expected
-Initiate logout processing via a Logout message with an appropriate error message, then waits for a confirming Logout before disconnecting if the Logon sequence number is less than expected. If the confirming Logout has not been received within a short period of time the session will be disconnected.
-Handle retransmission requests.
-Initiate a Logon using the SenderCompID in the message header.
-Forwarded to the FIX client messages that are waiting in the outbound queue
-Begin regular message communication.
<br/>

| Tag  | Field Name              | Required  | Note |
| ---- | ----------------------- | --------  | ---- |
|      | Standard Header         | Yes       | MsgType = A  |
| 98   | EncryptMethod           | Yes       | 0 = NONE_OTHER <br/> 1 = PKCS <br/> 2 = DES <br/> 3 = PKCS_DES <br/> 4 = PGP_DES <br/> 5 = PGP_DES_MD5<br/> 6 = PEM_DES_MD5 |
| 108  | HeartBtInt              | Yes       |  |
| 141  | ResetSeqNumFlag         | No        |  |
| 1137 | DefaultApplVerID        | Yes       |  |
|      | Standard Trailer        | Yes       |  |

# Logout (5)

```fix
Example Message


Client
8  =  FIXT.1.1
9  =  53
35 =  5
34 =  6
49 =  YOURSENDERCOMP
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

# Heartbeat (0)

```fix
Example Message

Client
8  =  FIXT.1.1
9  =  57
35 =  0
34 =  841
49 =  YOURSENDERCOMP
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
56 =  YOURSENDERCOMP
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

# <br/>
# FIX ORDER ENTRY MESSAGES
The Zero Hash Matching Engine will respond to any new order messages with execution reports. Example order messages and execution report responses are available below.
<br/>
This section outlines the FIX messages, how they are supported, and to what extent the business data is translated within the FIX Gateway.
<br/>

# New Order - Single (D)

```fix
Example Message

Client - BTC/USD Buy Limit GTC Order for OrderQty 0.01 at Price of 19000
8   =   FIXT.1.1
9   =   254
35  =   D
34  =   426
49  =   YOURSENDERCOMP
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
56  =  YOURSENDERCOMP
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
56  =   YOURSENDERCOMP
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
| 1   | Account          | Yes      | Account created in matching engine in format : firms/<Clearing Party Name>/accounts/<Account Name>                                                                   |
| 21  | HandlInst        | Yes      | '1' = Automated execution order, private, no Broker intervention '2' = Automated execution order, public, Broker intervention OK  '3' = Manual order, best execution |
| 55  | Symbol           | Yes      | Symbol of instrument                                                                                                                                                 |
| 48  | SecurityID       | Yes      | Security ID of instrument, usually same as symbol                                                                                                                    |
| 54  | Side             | Yes      | '1' = Buy '2' = Sell                                                                                                                                                 |
| 38  | OrderQty         | Yes      | Size of the order. E.G. 10                                                                                                                                           |
| 40  | OrdType          | Yes      | '2' = Limit <br/> '3' = Stop <br/> '4' = Stop Limit <br/> 'K' = Market with left over as Limit                                                                       |
| 44  | Price            | No       | Required for limit Ordtypes                                                                                                                                          |
| 50  | User             | Yes      | User created with specific relationships in matching engine                                                                                                          |
| 59  | TimeInForce      | No       | 0 = DAY <br/> 1 = GOOD_TILL_CANCEL <br/> 3 = IMMEDIATE_OR_CANCEL <br/> 6 = GOOD_TILL_DATE                                                                            |
| 60  | TransactTime     | Yes      | Time of execution/order creation (UTC Time in datetime format)                                                                                                       |
| 126 | ExpireTime       | No       | Required if TimeInForce = 6 (UTC Time in datetime format - 17:18:02.54)                                                                                              |
|     | Instrument       | Yes      | Component Fields can be found in appendix                                                                                                                            |
|     | Standard Trailer | Yes      |                                                                                                                                                                      |

# Order Cancel Request (F)

```fix
Example Message

Client
8    =    FIXT.1.1
9    =    238
35   =    F
34   =    76
49   =    YOURSENDERCOMP
50   =    trader
52   =    20220817-13:02:41.202
56   =    ZERO
1128 =    9
1    =    firms/Zero-Hash-Liquidity/accounts/test
11   =    16329117-3bf0-4684-b74a-54f44adbbe84
38   =    0.02
41   =    e31e8ba5-2759-4255-8f08-91c1d18b9ebb
54   =    1
55   =    BTC/USD
60   =    20220817-13:02:41
10   =    171
```

| Tag | Field Name       | Required | Note                                                                                         |
| --- | ---------------- | -------- | -------------------------------------------------------------------------------------------- |
|     | Standard Header  | Yes      |                                                                                              |
| 41  | OrigClOrdId      | Yes      | ClOrdID of the previous order                                                                |
| 37  | OrdId            | No       | Unique Identifier of order assigned by the platform.                                         |
| 11  | ClOrdId          | Yes      | Unique identifier of the order. Must be unique for each session, max 32 chars.               |
| 60  | TransactTime     | Yes      | Time of execution/order creation (UTC Time in datetime format)                               |
| 55  | Symbol           | Yes      | Common, “human understood” representation of the security, bitcoins US Dollar – BTCUSD, etc. |
| 54  | Side             | Yes      | '1' = Buy '2' = Sell                                                                         |
| 38  | OrderQty         | Yes      | Size of the order. E.G. 10                                                                   |
| 58  | Text             | No       |                                                                                              |
|     | Standard Trailer | Yes      |                                                                                              |



# Order Cancel/Replace Request (G)

```fix
Example message

Client
8    =    FIXT.1.1
9    =    294
35   =    G
34   =    32
49   =    YOURSENDERCOMP
50   =    trader
52   =    20220818-10:02:48.403
56   =    ZERO
1128 =    9
1    =    firms/Zero-Hash-Liquidity/accounts/test
11   =    b0328323-0daa-4654-aa9d-b0a9a77af8e0
21   =    1
38   =    0.03
40   =    2
41   =    84dae91a-50a6-44f1-8c9f-6fd51ca46560
44   =    93500
54   =    1
55   =    BTC/USD
59   =    1
60   =    20220818-10:02:48
453  =    1
448  =    DBS_GROUP
447  =    D
452  =    1
10   =    098

Gateway
8   =   FIXT.1.1
9   =   410
35  =   8
34  =   67
49  =   ZERO
52  =   20220818-10:02:48.452815018
56  =   FUTU-1
57  =   TRADER
1   =   firms/Zero-Hash-Liquidity/accounts/FUTU
6   =   0.000000000
11  =   b0328323-0daa-4654-aa9d-b0a9a77af8e0
14  =   0.00
17  =   1560205564477079552
22  =   8
31  =   0.00
32  =   0.00
37  =   1560205405594038272
38  =   0.03
39  =   0
40  =   2
41  =   84dae91a-50a6-44f1-8c9f-6fd51ca46560
44  =   93500.00
48  =   BTC/USD
54  =   1
55  =   BTC/USD
59  =   1
60  =   20220818-10:02:48.445059165
99  =   0.00
150 =   5
151 =   0.03
581 =   14
582 =   4
10  =   143
```
The Execution Report is automatically generated and sent by the gatway automatically.

| Tag | Field Name       | Required | Note                                                                                           |
| --- | ---------------- | -------- | ---------------------------------------------------------------------------------------------- |
|     | Standard Header  | Yes      |                                                                                                |
| 41  | OrigClOrdId      | Yes      | ClOrdID of the previous order                                                                  |
| 37  | OrdId            | No       | Unique Identifier of order assigned by the platform.                                           |
| 11  | ClOrdId          | Yes      | Unique identifier of the order. Must be unique for each session, max 32 chars.                 |
| 1   | Account          | No       | Optional identifier from customer, will be passed back in Execution Report.                    |
| 55  | Symbol           | Yes      | Common, “human understood” representation of the security, bitcoins US Dollar – BTCUSD, etc.   |
| 54  | Side             | Yes      | '1' = Buy '2' = Sell                                                                           |
|     | OrderQtyData     | Yes      | Component fields can be found in appendix                                                      |
| 40  | OrdType          | Yes      | '2' = Limit <br/> '3' = Stop <br/> '4' = Stop Limit <br/> 'K' = Market with left over as Limit |
| 44  | Price            | No       | Required for limit Ordtypes                                                                    |
| 59  | TimeInForce      | No       | '3' = IOC (Immediate-Or-Cancel)                                                                |
| 126 | ExpireTime       | No       | Required if TimeInForce = GTT (UTC time in datetime format - 17:18:02.544)                     |
|     | Instrument       | Yes      | Component Fields can be found in appendix                                                      |
|     | Standard Trailer | Yes      |                                                                                                |

# Execution Report (8)

The Execution Report is automatically generated and sent by the FIX Gateway .


| Tag | Field Name       | Required | Note                                                                                                                                                   |
| --- | ---------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
|     | Standard Header  | -        |                                                                                                                                                        |
| 37  | OrdId            | -        | Unique Identifier of order assigned by the platform.                                                                                                   |
| 6   | AvgPx            | -        |                                                                                                                                                        |
| 11  | ClOrdId          | -        | Unique identifier of the order copied from customer's request.                                                                                         |
| 41  | OrigClOrdId      | -        | ClOrdID of the previous order                                                                                                                          |
| 14  | CumQty           | -        |                                                                                                                                                        |
| 17  | ExecID           | -        | Platform assigned execution ID                                                                                                                         |
| 20  | ExecTransType    | -        | 0                                                                                                                                                      |
| 32  | LastQty          | -        |                                                                                                                                                        |
| 37  | OrderID          | -        |                                                                                                                                                        |
| 38  | OrderQty         | -        | Size of the order. E.G. 10                                                                                                                             |
| 39  | OrdStatus        | -        | 0 = New, 1 = PartiallyFilled, 2 = Filled, 4 = Canceled, 5 = Replaced, 6 = PendingCancel, 8 = Rejected, A = PendingNew, C = Expired, E = PendingReplace |
| 40  | OrdType          | -        | '2' = Limit <br/> '3' = Stop <br/> '4' = Stop Limit <br/> 'K' = Market with left over as Limit                                                         |
| 44  | Price            | -        |                                                                                                                                                        |
| 54  | Side             | -        | '1' = Buy  '2' = Sell                                                                                                                                  |
| 55  | Symbol           | -        | Common, “human understood” representation of the trading pair, bitcoins US Dollar – BTCUSD, etc.                                                       |
| 59  | TimeInForce      | -        |                                                                                                                                                        |
| 60  | TransactTime     | -        |                                                                                                                                                        |
| 126 | ExpireTime       | -        | Time order will expire                                                                                                                                 |
| 150 | ExecType         | -        |                                                                                                                                                        |
| 151 | LeavesQty        | -        |                                                                                                                                                        |
| 58  | Text             | -        |                                                                                                                                                        |
|     | Standard Trailer | -        |                                                                                                                                                        |

# Order Cancel Reject (9)
This message is send when a Order Cancel Request or Order Cancel/Replace Request is rejected

| Tag | Field Name       | Required | Note                                                                           |
| --- | ---------------- | -------- | ------------------------------------------------------------------------------ |
|     | Standard Header  | -        |                                                                                |
| 41  | OrigClOrdId      | -        | ClOrdID of the previous order.                                                 |
| 37  | OrdId            | -        | Unique Identifier of order assigned by the platform.                           |
| 11  | ClOrdId          | -        | Unique identifier of the order. Must be unique for each session, max 32 chars. |
| 39  | OrdStatus        | -        | OrdStatus value after this cancel reject is applied.                           |
| 102 | CxlRejReason     | -        | Code to identify reason for cancel rejection.                                  |
| 58  | Text             | -        | Optional explanation message.                                                  |
| 434 | CxlRejResponseTo | -        | Unique identifier of the Canceled order.                                       |
|     | Standard Trailer | -        |                                                                                |


# <br/>

# FIX MARKET DATA MESSAGES

The information in Zero Hash FIX API specification describes the adaptation of the standard FIX 5.0 for vendors and subscribers to communicate with the Zero Hash market. FIX 5.0 tags, as described in detail on the Financial Information Exchange Protocol Committee website, www.fixprotocol.org as well as custom tags are used extensively in this document and the reader should familiarize themselves with the latest updates to this release. If an application message in Financial Information Exchange Protocol version 5.0, or previous FIX versions, is not included in this document, the message is ignored.

# Market Data Request (V)

```fix
Example FIX Message

The FIX message would appear as 

8   =   FIXT.1.1
9   =   133
35  =   V
34  =   306
49  =   YOURSENDERCOMP
52  =   20220915-09:58:59.011
56  =   ZERO
146 =   1
55  =   BTC/USD
48  =   BTC/USD
22  =   8
262 =   ZERO
263 =   1
264 =   2
267 =   3
269 =   0
269 =   1
269 =   2
10  =   093
```

| Tag | Field Name              | Required | Note                                                                                                 |
| --- | ----------------------- | -------- | ---------------------------------------------------------------------------------------------------- |
|     | Standard Header         |          | MsgType = W                                                                                          |
| 262 | MDReqID                 |          | The MDReqID of the MarketDataRequest message.                                                        |
| 55  | Symbol                  |          | Identifier for the symbol                                                                            |
| 48  | SecurityID              |          | Security ID (usually same as symbol)                                                                 |
| 269 | MDEntryType             |          | Type of market data entry. Valid values: 0 = Bid, 1 = Offer, 2 = Trade                               |
| 263 | SubscriptionRequestType |          | 0 = SNAPSHOT <br/> 1 = SNAPSHOT_PLUS_UPDATES <br/> 2 = DISABLE_PREVIOUS_SNAPSHOT_PLUS_UPDATE_REQUEST |
|     | Standard Trailer        |          |                                                                                                      |


# Market Data Responses – Top of Book Snapshot (W)

```fix
Example FIX Message

The FIX message would appear as 

8    =  FIXT.1.1
9    =  1187
35   =  W
34   =  8819
49   =  ZERO
52   =  20220915-09:58:59.131087940
56   =  YOURSENDERCOMP
22   =  8
48   =  BTC/USD
55   =  BTC/USD
167  =  NONE
262  =  ZERO
268  =  11
269  =  0
270  =  20000.00
271  =  0.01000000
272  =  20220915
273  =  09:58:58.701896604
59   =  1
37   =  1570062592651415552
278  =  1570062592651415552
40   =  2
269  =  0
270  =  19931.67
271  =  0.00775000
272  =  20220915
273  =  09:58:58.701896604
59   =  1
37   =  1570351436330594304
278  =  1570351436330594304
40   =  2
269  =  0
270  =  19873.28
271  =  0.01550000
272  =  20220915
273  =  09:58:58.701896604
59   =  1
37   =  1570351460967936000
278  =  1570351460967936000
40   =  2
269  =  1
270  =  20335.34
271  =  0.00625000
272  =  20220915
273  =  09:58:58.701896604
59   =  1
37   =  1570351423974174720
278  =  1570351423974174720
40   =  2
269  =  1
270  =  20393.72
271  =  0.01250000
272  =  20220915
273  =  09:58:58.701896604
59   =  1
37   =  1570351448670236672
278  =  1570351448670236672
40   =  2
269  =  2
270  =  44000.00
271  =  0.02500000
272  =  20220914
273  =  18:12:59.695310898
336  =  OPEN
269  =  4
270  =  20523.20
272  =  20220914
273  =  12:44:19.851431213
336  =  OPEN 0    
1070 =  1
269  =  6
270  =  20318.00
272  =  20220914
273  =  12:44:03.897119444
336  =  OPEN
269  =  7
270  =  44000.00
272  =  20220914
273  =  18:12:59.695310898
336  =  OPEN
269  =  8
270  =  20333.32
272  =  20220914
273  =  14:31:11.180821686
336  =  OPEN
269  =  B
270  =  53138.0442000000
271  =  2.43500000
272  =  20220914
273  =  18:12:59.695310898
336  =  OPEN 1     
1151 =  BTC
10   =  238
```

| Tag | Field Name       | Required | Note                                                        |
| --- | ---------------- | -------- | ----------------------------------------------------------- |
|     | Standard Header  |          | MsgType = W                                                 |
| 262 | MDReqID          |          | The MDReqID of the MarketDataRequest message.               |
| 55  | Symbol           |          | Identifier for the symbol                                   |
| 269 | MDEntryType      |          | Type of market data entry. Valid values: 0 = Bid, 1 = Offer |
| 270 | MDEntryPx        |          | Price of the market data entry.                             |
| 271 | MDEntrySize      |          | Qty represented by the Market Data Entry.                   |
|     | Standard Trailer |          |                                                             |



# Market Data Responses– Incremental Refresh (X)

```fix
Example FIX Message

The FIX message would appear as 

8    =    FIXT.1.1
9    =    256
35   =    X
34   =    12781
49   =    ZERO
52   =    20220915-12:55:24.957686955
56   =    YOURSENDERCOMP
262  =    ZERO
268  =    1
279  =    0
269  =    0
278  =    1570395862859935744
55   =    BTC/USD
48   =    BTC/USD
22   =    8
167  =    NONE
1151 =    BTC
270  =    19609.77
271  =    0.05425000
272  =    20220915
273  =    12:55:24.937078681
59   =    1
37   =    1570395862859935744
40   =    2
10   =    049
```
<br/>

| Tag | Field Name       | Required | Note                                                                                                                                  |
| --- | ---------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------- |
|     | Standard Header  |          | MsgType = X                                                                                                                           |
| 262 | MDReqID          |          | The MDReqID of the MarketDataRequest message.                                                                                         |
| 279 | MDUpdateAction   |          | The Market Data update action type. It must be the first field in the repeating group. The only valid values are: 0 = New, 2 = Delete |
| 269 | MDEntryType      |          | Type of market data entry. Valid values: 0 = Bid, 1 = Offer, 2=Trade                                                                  |
| 278 | MDEntryID        |          | Market data identifier                                                                                                                |
| 55  | Symbol           |          | Identifier for the symbol                                                                                                             |
| 48  | SecurityID       |          | Security ID                                                                                                                           |
| 270 | MDEntryPx        |          | Price of the market data entry.                                                                                                       |
| 271 | MDEntrySize      |          | Number of shares represented by the Market Data Entry.                                                                                |
| 58  | Text             |          | Order ID reference                                                                                                                    |
|     | Standard Trailer |          |                                                                                                                                       |

# Security List Request (x)

```fix
Example FIX Message

The FIX message would appear as 

8   =   FIXT.1.1
9   =   68
35  =   x
34  =   2
49  =   FUTU-2
52  =   20220914-10:02:28.627
56  =   ZERO
320 =   FUTU
559 =   4
10  =   128
```
<br/>

| Tag | Field Name              | Required | Note                                                                                                                                                                       |
| --- | ----------------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|     | Standard Header         |          | MsgType = x                                                                                                                                                                |
|     | Instrument              |          | Instrument to be retrieved using fields 55 & 48 if empty will retrieve list of everything                                                                                  |
| 559 | SecurityListRequestType |          | 0 = SYMBOL <br/> 1 = SECURITYTYPE_AND_OR_CFICODE <br/> 2 = PRODUCT <br/> 3 = TRADINGSESSIONID <br/> 4 = ALL_SECURITIES <br/> 5 = MARKETID_OR_MARKETID_PLUS_MARKETSEGMENTID |
| 320 | SecurityReqID           |          | Request ID                                                                                                                                                                 |
| 58  | Text                    |          | Text                                                                                                                                                     |
|     | Standard Trailer        |          |                                                                                                                                                                            |


# Business Message Reject (j)

<br/>

| Tag | Field Name           | Required | Note                                                                                                                                                                                                                                                                                            |
| --- | -------------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|     | Standard Header      |          | MsgType = j                                                                                                                                                                                                                                                                                     |
| 380 | BusinessRejectReason |          | 0 = OTHER <br/> 1 = UNKNOWN_ID <br/> 18 = INVALID_PRICE_INCREMENT <br/> 2 = UNKNOWN_SECURITY <br/> 3 = UNSUPPORTED_MESSAGE_TYPE <br/> 4 = APPLICATION_NOT_AVAILABLE <br/> 5 = CONDITIONALLY_REQUIRED_FIELD_MISSING <br/> 6 = NOT_AUTHORIZED <br/> 7 = DELIVERTO_FIRM_NOT_AVAILABLE_AT_THIS_TIME |
| 379 | BusinessRejectRefID  |          | Rejection ID                                                                                                                                                                                                                                                                                    |
| 372 | RefMsgType           |          | Reference Message type                                                                                                                                                                                                                                                                          |
| 45  | RefSeqNum            |          | Reference Sequence Number                                                                                                                                                                                                                                                                       |
|     | Standard Trailer     |          |                                                                                                                                                                                                                                                                                                 |
# <br/>
# APPENDIX

Below is the appendix for needed components included in Order Executions & Market Data requests:

## Instrument

| Tag | Field Name        | Required | Data Type    | Accepted Enums                                                                                                                                                                                                                                  |
| --- | ----------------- | -------- | ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 231 | ContactMultiplier |          | FLOAT        |                                                                                                                                                                                                                                                 |
| 541 | MaturityDate      |          | LOCALMKTDATE |                                                                                                                                                                                                                                                 |
| 969 | MinPriceIncrement |          | FLOAT        |                                                                                                                                                                                                                                                 |
| 460 | Product           |          | INT          | 1 = AGENCY <br/> 10 = MORTGAGE <br/> 11 = MUNICIPAL <br/> 12 = OTHER <br/> 13 = FINANCING <br/> 2 = COMMODITY <br/> 3 = CORPORATE <br/> 4 = CURRENCY <br/> 5 = EQUITY <br/> 6 = GOVERNMENT <br/> 7 = INDEX <br/> 8 = LOAN <br/> 9 = MONEYMARKET |
| 48  | SecurityID        |          | STRING       |                                                                                                                                                                                                                                                 |
| 22  | SecurityIDSource  |          | STRING       | 8=Exchange Symbol (Currently this is only used for Crypto)                                                                                                                                                                                      |
| 762 | SecuritySubType   |          | STRING       |                                                                                                                                                                                                                                                 |
| 167 | SecurityType      |          | STRING       | CS = COMMON_STOCK (Currently this is only used for Crypto)                                                                                                                                                                                      |
| 55  | Symbol            |          | STRING       |                                                                                                                                                                                                                                                 |

## OrderQtyData

| Tag | Field Name   | Required | Data Type | Accepted Enums |
| --- | ------------ | -------- | --------- | -------------- |
| 152 | CashOrderQty |          | QTY       |                |
| 38  | OrderQty     |          | QTY       |                |

## Parties

| Tag  | Field Name    | Required | Data Type | Accepted Enums                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ---- | ------------- | -------- | --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|      | NoPartyIDs    |          |           |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| >448 | PartyID       |          | STRING    |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| >447 | PartyIDSource |          | CHAR      | <br/> 1 = KOREAN_INVESTOR_ID <br/> 2 = TAIWANESE_QUALIFIED_FOREIGN_INVESTOR_ID_QFII_FID <br/> 3 = TAIWANESE_TRADING_ACCT <br/> 4 = MALAYSIAN_CENTRAL_DEPOSITORY <br/> 5 = CHINESE_INVESTOR_ID <br/> 6 = UK_NATIONAL_INSURANCE_OR_PENSION_NUMBER <br/> 7 = US_SOCIAL_SECURITY_NUMBER <br/> 8 = US_EMPLOYER_OR_TAX_ID_NUMBER <br/> 9 = AUSTRALIAN_BUSINESS_NUMBER <br/> A = AUSTRALIAN_TAX_FILE_NUMBER <br/> B = BIC <br/> C = GENERALLY_ACCEPTED_MARKET_PARTICIPANT_IDENTIFIER <br/> D = PROPRIETARY <br/> E = ISO_COUNTRY_CODE <br/> F = SETTLEMENT_ENTITY_LOCATION <br/> G = MIC <br/> H = CSD_PARTICIPANT_MEMBER_CODE <br/> I = DIRECTED_BROKER_THREE_CHARACTER_ACRONYM_AS_DEFINED_IN_ISITC_ETC_BEST_PRACTICE_GUIDELINES_DOCUMENT <br/> N = LEGAL_ENTITY_IDENTIFIER |
| >452 | PartyRole     |          | INT       | 24 = Customer Account                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |

## CommissionData

| Tag | Field Name | Required | Data Type | Accepted Enums                                                                                                                                                               |
| --- | ---------- | -------- | --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 13  | CommType   |          | CHAR      | 1 = PER_UNIT <br/> 2 = PERCENT <br/> 3 = ABSOLUTE <br/> 4 = PERCENTAGE_WAIVED_CASH_DISCOUNT <br/> 5 = PERCENTAGE_WAIVED_ENHANCED_UNITS <br/> 6 = POINTS_PER_BOND_OR_CONTRACT |
| 12  | Commission |          | FLOAT     |                                                                                                                                                                              |

## InstrmtLegExecGrp

| Tag   | Field Name    | Required | Data Type | Accepted Enums |
| ----- | ------------- | -------- | --------- | -------------- |
|       | NoLegs        |          |           |                |
| >     | InstrumentLeg |          | Component |                |
| >637  | LegLastPx     |          | PRICE     |                |
| >1418 | LegLastQty    |          | QTY       |                |

## InstrumentLeg

| Tag | Field Name          | Required | Data Type | Accepted Enums |
| --- | ------------------- | -------- | --------- | -------------- |
| 566 | LegPrice            |          | PRICE     |                |
| 623 | LegRatioQty         |          | FLOAT     |                |
| 602 | LegSecurityID       |          | STRING    |                |
| 603 | LegSecurityIDSource |          | STRING    |                |
| 764 | LegSecuritySubType  |          | STRING    |                |
| 609 | LegSecurityType     |          | STRING    |                |
| 624 | LegSide             |          | CHAR      |                |
| 600 | LegSymbol           |          | STRING    |                |

## EventGrp

| Tag   | Field Name | Required | Data Type    | Accepted Enums                                                           |
| ----- | ---------- | -------- | ------------ | ------------------------------------------------------------------------ |
|       | NoEvents   |          |              |                                                                          |
| >866  | EventDate  |          | LOCALMKTDATE |                                                                          |
| >848  | EventText  |          | STRING       |                                                                          |
| >1145 | EventTime  |          | UTCTIMESTAMP |                                                                          |
| >865  | EventType  |          | INT          | 5 = ACTIVATION <br/> 6 = INACTIVATION <br/> 7 = LAST_ELIGIBLE_TRADE_DATE |

## MDFullGrp

| Tag   | Field Name         | Required | Data Type         | Accepted Enums                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ----- | ------------------ | -------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
|       | NoMDEntries        |          |                   |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| >15   | Currency           |          | CURRENCY          |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| >18   | ExecInst           |          | MULTIPLECHARVALUE | 0 = STAY_ON_OFFER_SIDE <br/> 1 = NOT_HELD <br/> 2 = WORK <br/> 3 = GO_ALONG <br/> 4 = OVER_THE_DAY <br/> 5 = HELD <br/> 6 = PARTICIPANT_DONT_INITIATE <br/> 7 = STRICT_SCALE <br/> 8 = TRY_TO_SCALE <br/> 9 = STAY_ON_BID_SIDE <br/> A = NO_CROSS <br/> B = OK_TO_CROSS <br/> C = CALL_FIRST <br/> D = PERCENT_OF_VOLUME <br/> E = DO_NOT_INCREASE <br/> F = DO_NOT_REDUCE <br/> G = ALL_OR_NONE <br/> H = REINSTATE_ON_SYSTEM_FAILURE <br/> I = INSTITUTIONS_ONLY <br/> J = REINSTATE_ON_TRADING_HALT <br/> K = CANCEL_ON_TRADING_HALT <br/> L = LAST_PEG <br/> M = MID_PRICE_PEG <br/> N = NON_NEGOTIABLE <br/> O = OPENING_PEG <br/> P = MARKET_PEG <br/> Q = CANCEL_ON_SYSTEM_FAILURE <br/> R = PRIMARY_PEG <br/> S = SUSPEND <br/> T = FIXED_PEG_TO_LOCAL_BEST_BID_OR_OFFER_AT_TIME_OF_ORDER <br/> U = CUSTOMER_DISPLAY_INSTRUCTION <br/> V = NETTING <br/> W = PEG_TO_VWAP <br/> X = TRADE_ALONG <br/> Y = TRY_TO_STOP <br/> Z = CANCEL_IF_NOT_BEST <br/> a = TRAILING_STOP_PEG <br/> b = STRICT_LIMIT <br/> c = IGNORE_PRICE_VALIDITY_CHECKS <br/> d = PEG_TO_LIMIT_PRICE <br/> e = WORK_TO_TARGET_STRATEGY <br/> f = INTERMARKET_SWEEP <br/> g = EXTERNAL_ROUTING_ALLOWED <br/> h = EXTERNAL_ROUTING_NOT_ALLOWED <br/> i = IMBALANCE_ONLY <br/> j = SINGLE_EXECUTION_REQUESTED_FOR_BLOCK_TRADE <br/> k = BEST_EXECUTION <br/> l = SUSPEND_ON_SYSTEM_FAILURE <br/> m = SUSPEND_ON_TRADING_HALT <br/> n = REINSTATE_ON_CONNECTION_LOSS <br/> o = CANCEL_ON_CONNECTION_LOSS <br/> p = SUSPEND_ON_CONNECTION_LOSS <br/> q = RELEASE_FROM_SUSPENSION <br/> r = EXECUTE_AS_DELTA_NEUTRAL_USING_VOLATILITY_PROVIDED <br/> s = EXECUTE_AS_DURATION_NEUTRAL <br/> t = EXECUTE_AS_FX_NEUTRAL |
| >126  | ExpireTime         |          | UTCTIMESTAMP      |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| >332  | HighPx             |          | PRICE             |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| >31   | LastPx             |          | PRICE             |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| >283  | LocationID         |          | STRING            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| >333  | LowPx              |          | PRICE             |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| >272  | MDEntryDate        |          | UTCDATEONLY       |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| >278  | MDEntryID          |          | STRING            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| >270  | MDEntryPx          |          | PRICE             |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| >271  | MDEntrySize        |          | QTY               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| >273  | MDEntryTime        |          | UTCTIMEONLY       |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| >269  | MDEntryType        |          | CHAR              | 0 = BID <br/> 1 = OFFER <br/> 2 = TRADE <br/> 3 = INDEX_VALUE <br/> 4 = OPENING_PRICE <br/> 5 = CLOSING_PRICE <br/> 6 = SETTLEMENT_PRICE <br/> 7 = TRADING_SESSION_HIGH_PRICE <br/> 8 = TRADING_SESSION_LOW_PRICE <br/> 9 = TRADING_SESSION_VWAP_PRICE <br/> A = IMBALANCE <br/> B = TRADE_VOLUME <br/> C = OPEN_INTEREST <br/> D = COMPOSITE_UNDERLYING_PRICE <br/> E = SIMULATED_SELL_PRICE <br/> F = SIMULATED_BUY_PRICE <br/> G = MARGIN_RATE <br/> H = MID_PRICE <br/> J = EMPTY_BOOK <br/> K = SETTLE_HIGH_PRICE <br/> L = SETTLE_LOW_PRICE <br/> M = PRIOR_SETTLE_PRICE <br/> N = SESSION_HIGH_BID <br/> O = SESSION_LOW_OFFER <br/> P = EARLY_PRICES <br/> Q = AUCTION_CLEARING_PRICE <br/> R = DAILY_VALUE_ADJUSTMENT_FOR_LONG_POSITIONS <br/> S = SWAP_VALUE_FACTOR <br/> T = CUMULATIVE_VALUE_ADJUSTMENT_FOR_LONG_POSITIONS <br/> U = DAILY_VALUE_ADJUSTMENT_FOR_SHORT_POSITIONS <br/> V = CUMULATIVE_VALUE_ADJUSTMENT_FOR_SHORT_POSITIONS <br/> W = FIXING_PRICE <br/> X = CASH_RATE <br/> Y = RECOVERY_RATE <br/> Z = RECOVERY_RATE_FOR_LONG <br/> a = RECOVERY_RATE_FOR_SHORT                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| >1070 | MDQuoteType        |          | INT               | 0 = INDICATIVE<br/> 1 = TRADEABLE<br/> 2 = RESTRICTED_TRADEABLE<br/> 3 = COUNTER<br/> 4 = INDICATIVE_AND_TRADEABLE                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| >110  | MinQty             |          | QTY               |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| >286  | OpenCloseSettlFlag |          | MULTIPLECHARVALUE | 0 = DAILY_OPEN <br/> 1 = SESSION_OPEN <br/> 2 = DELIVERY_SETTLEMENT_ENTRY <br/> 3 = EXPECTED_ENTRY <br/> 4 = ENTRY_FROM_PREVIOUS_BUSINESS_DAY <br/> 5 = THEORETICAL_PRICE_VALUE                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| >40   | OrdType            |          | CHAR              | 2 = LIMIT <br/> 3 = STOP <br/> 4 = STOP_LIMIT <br/> K = MARKET_WITH_LEFT_OVER_AS_LIMIT                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| >37   | OrderID            |          | STRING            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| >     | Parties            |          | Component         |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| >762  | SecuritySubType    |          | STRING            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| >58   | Text               |          | STRING            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| >59   | TimeInForce        |          | CHAR              | 0 = DAY <br/> 1 = GOOD_TILL_CANCEL <br/> 3 = IMMEDIATE_OR_CANCEL <br/> 6 = GOOD_TILL_DATE                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| >336  | TradingSessionID   |          | STRING            |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| >828  | TrdType            |          | INT               | 0 = REGULAR_TRADE (Default)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |

# <br/>