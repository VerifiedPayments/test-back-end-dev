# Backend Challenge for Developer Candidates

## Project description

We want you to develop a small application that allows sending money using 3'rd party service. Application needs to be a REST API.

* Application must have 4 endpoints:
  * To add transaction (POST [content body: SWIFT MT103 message])
  * List all transactions (GET with filtering posibility (sender_id, transaction_id, recipient, amount, currency, processed))
  * To process transaction (PUT [uri with: transaction_id])
  * To cancel transaction (DELETE [uri with: transaction_id])

### Api functionality

It can support json, xml or form-data. You can choose any format you prefer. It could be done using any framework but Phalcon is preferred. Make README.md file and outline database structure and indexes used and describe why you chose such implementation of 'Process transaction' and why those responses for client and what should be made differently (You don't have to make it 100% bullet proof you just can make basics of it and outline what and why should be made more (There is no difference if you make full code or explain in details it's full functionality and do just basic code - your choice it's just for your time saving)). Document those 4 api endpoints using http://apidocjs.com/. Log all api requests to database (just time, uri, method [POST, GET, PUT, DELETE], status [success OR error])

### Add transaction functionality

It should store message content to database from SWIFT MT103 message (sender_id, transaction_id, recipient, amount, currency, details). Where those fields would be (In reality they are different):
* sender_id - value from 1 field (F01AAAAGRA0AXXX0057000289)
* transaction_id - value from 2 field (O1030919010321BBBBGRA0AXXX00570001710103210920N)
* recipient - value from 4 field with key 20 (5387354)
* currency - value from 4 field with key 32A 3 characters after date (first 6 digits) (USD)
* amount - value from 4 field with key 32A characters after currency (1101,)
* details - json with rest values from field 4 (some values can be arrays like (71F), and some values can be multiline (for example - 50A) those new lines will never start with: but can contain it)

And it should return success or error if it failed to parse message or one of those 6 fields was missing, or it's duplicate message (sender_id, transaction_id together they are unique). You will find examples of SWIFT MT103 messages on bottom

### List all transactions functionality

It should return list of transactions in format you prefer most (json, xml, or anything else)

### Process transaction functionality

You should make service that will call - https://verifiedpayments.com/test.php
* POST (For transaction creation) with json fields recipient, amount, currency - it will return transaction identifier (it's dummy random now but imagine that it's unique) on success or error if something happened, although it can return malicious json or die. 
```
POST https://verifiedpayments.com/test.php
content-type: application/json

{
    "recipient" : "jonas",
    "amount": 20,
    "currency": "usd"
}
```
* GET (For transaction confirmation) with json field transaction - it will return message 'success' on success or error if something happened (already processed, insufficient funds, or something else), although it can return malicious json or die. (in last 2 cases it doesn't mean that transaction failed)
Have in mind that your application can fail for some reason beyond your control to and that this service is one of n and it can be changed by other easily. (And all possible cases you can imagine are available and communication with client is for your choose)
```
GET https://verifiedpayments.com/test.php
content-type: application/json

{
    "transaction": 54776
}
```
### Cancel transaction functionality

It should cancel unprocessed transaction (transactions that failed can be canceled to)

### SWIFT MT103 messages

```code
{1:F01AAAAGRA0AXXX0057000289}{2:O1030919010321BBBBGRA0AXXX00570001710103210920N}{4:
:20:5387354
:13C:/RNCTIME/1356+0000
:23B:CRED
:23E:PHOB/20.527.19.60
:26T:K90
:32A:000526USD1101,
:33B:USD1121,
:36:0,98766
:50A:/293456-1254349-82
VISTUS31
:50F:/12345678
1/SMITH JOHN
2/299, PARK AVENUE
3/US/NEW YORK, NY 10017
:50K:FRANZ HOLZAPFEL GMBH
VIENNA
:52A:BKAUATWW
:59:723491524
C. KLEIN
BLOEMENGRACHT 15
AMSTERDAM
:71A:SHA
:71F:USD10,
:71F:USD10,
:72:/INS/CHASUS33
-}{5:{MAC:75D138E4}{CHK:DE1B0D71FA96}}
```

```code
{1:F01MIDLGB22AXXX0548034693}{2:I103BKTRUS33XBRDN3}{3:{108:MT103}}{4: 
:20:8861198-0706 
:23B:CRED 
:32A:000612USD5443,99 
:33B:USD5443,99 
:50K:GIAN ANGELO IMPORTS 
NAPLES 
:52A:BCITITMM500 
:53A:BCITUS33 
:54A:IRVTUS3N 
:57A:BNPAFRPPGRE 
:59:/20041010050500001M02606 
KILLY S.A. 
GRENOBLE 
:70:/RFB/INVOICE 559661 
:71A:SHA 
:71F:USD10,
-}
```
