# NIBSS wallet API

The NIBSS wallet API enables developers to programatically access mobile money wallets.
This allows deveopers to create mobile money wallets as well as send and receive bank transfers via the Nigerian Interbank Settlment Systems.

This repository documents the NIBSS wallet API and contains bindings for the following languages/platforms
* [C#](https://github.com/cowriesys/nibss/tree/master/cs)
* [Java](https://github.com/cowriesys/nibss/tree/master/java)
* [Javascript](https://github.com/cowriesys/nibss/tree/master/js)
* [Python](https://github.com/cowriesys/nibss/tree/master/python)

To start using the API immeadiately, download the code samples for your chosen platform.
The remainder of this document provides the specification for the API and describes how it is implemented.

## API Structure
The NIBSS wallet API is based on a HTTP/REST architecture. API clients issue HTTP GET requests with parameters specified in the query string and HTTP POST requests with request body in JSON format. API responses use standard HTTP response codes with messages encoded in JSON format.

## API Security
Security for the API is enforced through a combination of SSL and HMAC256 signatures. API requests are only accepted over HTTPS.

## Request Authentication
API client requests are authenticated and authorized using a supplied ClientId and ClientKey. For every request, the API client must include the ClientId, Nonce and Signature HTTP headers and sign the request using the ClientKey.

## Computing the Signature
The algorithm used to compute the signature is described as follows 
1. Generate a nonce (it can be any unique string)
2. For HTTP POST requests, concatenate the nonce and the request body
`<nonce>{"phone": "08124661601", "first_name": "Wasiu", "last_name": "Ayinde"}`
3. Convert the base64 encoded ClientKey to bytes 
4. Instantiate a SHA256 object from the ClientKey bytes 
5. For HTTP GET requests, compute the SHA256 hash of the nonce. For HTTP POST requests, compute the SHA256 hash of the concatenated nonce and request body. The result yields the signature. Convert the signature to base64 format 
6. Apply the ClientId, Signature and Nonce as HTTP headers

## API Methods
Name|Description
----|-----------
[Create Account](#create-account-request)|Creates a new mobile money wallet
[Get Account](#get-account-request)|Retrieves a mobile money wallet account details 
[Name Enquiry](#name-enquiry-request)|Lookup the account name associated with a NUBAN bank account number
[Bank Transfer](#bank-transfer-request)|Initiates a bank transfer from a mobile money wallet to any bank account
[Check Transaction](#check-transaction-request)|Checks the status of a previously submitted transaction
[Get Transactions](#get-transactions-request)|Retrieves a list of transactions that occurred on a mobile money wallet
[Transaction Notification](#transaction-notification)|Receive a notification via a registered URL whenever a transaction occurs on a mobile money wallet

## Create Account Request
Request URL
```
POST https://nibss.cowriesys.com/api/account/create
```
Request Headers 
```
ClientId: c4091897667d44f4790674e37d9216453
Nonce: 20200410202513869
Signature: TAP2kgjhhodYUcawFIwsn2GSxjoyVvWWQDZMhHuMFFM=
```
Request Body
```javascript
{
    phone: "08124661601",
    first_name: "WASIU",
    last_name: "AYINDE"
}
```
Request Parameters
Name|Type|Description
----|----|-----------
phone|string|the user phone number identifying the mobile money account
first_name|string|the user first name
last_name|string|the user last name

## Create Account Response
A successful request will return the following JSON encoded response

**HTTP 200 OK**
```javascript
{
    id: "adc1760e0a86e4d82b387c71a92a8c0f3",
    account_name: "WASIU AYINDE",
    account_number: "08124661601",
    account_balance: "0.00",
    kyc: "3"
}
```
Response object fields
Name|Type|Description
----|----|-----------
id|string|unique id representing the mobile money account
account_name|string|the account name of the mobile money account
account_number|string|the account number of the mobile money account
account_balance|string|the current account balance of the mobile money account
kyc|string|the kyc level of the mobile money account 1, 2 or 3

## Get Account Request
Request URL
```
GET https://nibss.cowriesys.com/api/account/adc1760e0a86e4d82b387c71a92a8c0f3
```
Request Headers 
```
ClientId: c4091897667d44f4790674e37d9216453
Nonce: 20200410202513869
Signature: TAP2kgjhhodYUcawFIwsn2GSxjoyVvWWQDZMhHuMFFM=
```

## Get Account Response
A successful request will return the following JSON encoded response

**HTTP 200 OK**
```javascript
{
    id: "adc1760e0a86e4d82b387c71a92a8c0f3",
    account_name: "Wasiu Ayinde",
    account_number: "08124661601",
    account_balance: "12000.00",
    kyc: "3",
}
```
Response object fields
Name|Type|Description
----|----|-----------
id|string|unique id representing the mobile money account
account_name|string|the account name of the mobile money account
account_number|string|the account number of the mobile money account
account_balance|string|the current account balance of the mobile money account
kyc|string|the kyc level of the mobile money account 1, 2 or 3

## Name Enquiry Request
Request URL
```
GET https://nibss.cowriesys.com/api/enquiry/name?bank_code=44&account_number=0005538936
```
Request Headers 
```
ClientId: c4091897667d44f4790674e37d9216453
Nonce: 20200410202513869
Signature: TAP2kgjhhodYUcawFIwsn2GSxjoyVvWWQDZMhHuMFFM=
```
Response Parameters
Name|Type|Description
----|----|-----------
bank_code|string|the id code of the destination bank
account_number|string|the account number at the destination bank

## Name Enquiry Response
A successful request will return the following JSON encoded response

**HTTP 200 OK**
```javascript
{
    account_number: "0006638936",
    account_name: "AKANDE ABASS"
}
```
Response object fields
Name|Type|Description
----|----|-----------
account_number|string|the destination account number
account_name|string|the account name associated with the destination account number

## Bank Transfer Request
Request URL
```
POST https://nibss.cowriesys.com/api/account/adc1760e0a86e4d82b387c71a92a8c0f3/transfer
```
Request Headers 
```
ClientId: c4091897667d44f4790674e37d9216453
Nonce: 20200410202513869
Signature: TAP2kgjhhodYUcawFIwsn2GSxjoyVvWWQDZMhHuMFFM=
```
Request Body
```javascript
{
    xref: "57b2a6bc60d14e45ac81bb6b8205da0f",
    bank_code: "44",
    account_number: "0006638936",
    amount: "1500.00",
    narration: "INVOICE 1005"
}
```
Request Parameters
Name|Type|Description
----|----|-----------
xref|string|a client generated string that uniquely references every transaction request. It cannot be repeated in a new request
bank_code|string|the id code of the destination bank
account_number|string|the destination account number
amount|string|the amount to transfer formatted as 2 decimal place number string
narration|string|a text string that will be attached to the transaction

## Bank Transfer Response
A successful request will return the following JSON encoded response

**HTTP 200 OK**
```javascript
{
    timestamp: "2020-04-20T11:23:20.249Z",
    id: "td7a9856dc5e846f8933f54dad74cf1aa",
    xref: "57b2a6bc60d14e45ac81bb6b8205da0f",
    sequence: "35",
    status: "ok",
    message: "SENT",
    amount: "1500.00",
    fee: "5.00",
    account_number: "08124661601",
    account_balance: "10495.00"
}
```
Response object fields
Name|Type|Description
----|----|-----------
timestamp|datetime|the time when this transactoin occurred in ISO datetime format
id|string|the unique string identifying this transaction
xref|string|the client generated string that uniquely references this transaction
sequence|string|the sequence number for this transaction
status|string|the system code indication the status of this transaction. Can be either "ok" or "error"
message|string|the system message providing information on the transaction
amount|string|the amount transferred formatted as 2 decimal place number string
fee|string|the fee charged for this transfer formatted as 2 decimal place number string
account_number|string|the destination account number
account_balance|string|the account balance of the mobile money account after the transaction

## Check Transaction Request
Request URL
```
GET https://nibss.cowriesys.com/api/transaction/57b2a6bc60d14e45ac81bb6b8205da0f
```
Request Headers 
```
ClientId: c4091897667d44f4790674e37d9216453
Nonce: 20200410202513869
Signature: TAP2kgjhhodYUcawFIwsn2GSxjoyVvWWQDZMhHuMFFM=
```

## Check Transaction Response
A successful request will return the following JSON encoded response

**HTTP 200 OK**
```javascript
{
    timestamp: "2020-04-20T11:23:20.249Z",
    id: "td7a9856dc5e846f8933f54dad74cf1aa",
    xref: "57b2a6bc60d14e45ac81bb6b8205da0f",
    sequence: "35",
    status: "ok",
    message: "SENT",
    amount: "1500.00"
}
```
Response object fields
Name|Type|Description
----|----|-----------
timestamp|datetime|the time when this transactoin occurred in ISO datetime format
id|string|the unique string identifying this transaction
xref|string|the client generated string that uniquely references this transaction
sequence|string|the sequence number for this transaction
status|string|the system code indication the status of this transaction. Can be either "ok" or "error"
message|string|the system message providing information on the transaction
amount|string|the amount of this transaction formatted as 2 decimal place number string

## Get Transactions Request
Request URL
```
GET https://nibss.cowriesys.com/api/account/adc1760e0a86e4d82b387c71a92a8c0f3/transactions
```
Request Headers 
```
ClientId: c4091897667d44f4790674e37d9216453
Nonce: 20200410202513869
Signature: TAP2kgjhhodYUcawFIwsn2GSxjoyVvWWQDZMhHuMFFM=
```

## Get Transactions Response
A successful request will return the following JSON encoded response

**HTTP 200 OK**
```javascript
{
    count: 2,
    transactions:
    [
        {
            timestamp: "2020-04-20T11:23:20.249Z",
            id: "td7a9856dc5e846f8933f54dad74cf1aa",
            xref: "57b2a6bc60d14e45ac81bb6b8205da0f",
            sequence: "35",
            type: "debit",
            status: "ok",
            message: "SENT",
            amount: "1500.00",
            narration: "INVOICE 1005"
        },
        {
            timestamp: "2020-04-20T11:23:20.249Z",
            id: "ta5837174a2d64855bdcdee19fc168be1",
            xref: "57b2a6bc60d14e45ac81bb6b8205da0f",
            sequence: "36",
            type: "debit",
            status: "ok",
            message: "SENT",
            amount: "5.00",
            narration: "FEE td7a9856dc5e846f8933f54dad74cf1aa"
        }
    ]
}
```
Response object fields
Name|Type|Description
----|----|-----------
count|number|the number of results contained in this response
transactions|array|an array containing a list of transaction objects
transaction|object|
timestamp|datetime|the time when this transactoin occurred in ISO datetime format
id|string|the unique string identifying this transaction
xref|string|the client generated string that uniquely references this transaction
sequence|string|the sequence number for this transaction
type|string|the type of this transaction. Can be either "debit" or "credit"
status|string|the system code indication the status of this transaction. Can be either "ok" or "error"
message|string|the system message providing information on the transaction
amount|string|the amount of this transaction formatted as 2 decimal place number string
narration|string|the text narration attached to this transaction

## Transaction Notification
The registered URL must be SSL/TLS and will receive an HTTP POST with request headers ClientId and Signature set to HMAC256 of request body.
The request body is in JSON format
```
POST https://example.com/notify
```
Request Headers 
```
ClientId: c4091897667d44f4790674e37d9216453
Signature: TAP2kgjhhodYUcawFIwsn2GSxjoyVvWWQDZMhHuMFFM=
```
Request Body
```javascript
{
    timestamp: "2020-04-20T11:23:20.249Z",
    id: "td7a9856dc5e846f8933f54dad74cf1aa",
    xref: "57b2a6bc60d14e45ac81bb6b8205da0f",
    sequence: "35",
    type: "debit",
    status: "ok",
    message: "SENT",
    amount: "1500.00",
    narration: "INVOICE 1005",
    source_bank: "1022",
    source_account: "08124661601",
    source_name: "WASIU AYINDE",
    destination_bank: "44",
    destination_account: "0006638936",
    destination_name: "ABASS AKANDE"
}
```
The registered URL is expected to respond to the HTTP POST with HTTP 200 OK


## Response Error Codes
A failed request will result in one of the following HTTP response error codes

HTTP Code|HTTP Status|Description
---------|-----------|------------
400|Bad Request|One or more request parameters is invalid
401|Unauthorized|Signature does not match or request failed authentication
402|Payment Required|Insufficient balance
403|Forbidden|One or more required headers are missing
404|Not Found|Item could not be found for retrieval
409|Conflict|Nonce has been used before 
500|Server Error|The server encountered an error 

Any error response will return more information in the response body
```javascript
{
    status: "error",
    message: "INVALID ACCOUNT"
}
```
