---
title: Payconiq Partner API Specification

language_tabs:
  - shell
  - javascript
  - php

toc_footers:
  - <a href='#'>Sign Up</a>
  - <a href='https://github.com/swarm1/slate'>Documentation on GitHub</a>

includes:
  - errors
  - where are includes

search: true
---

# Summary

Partner API Specification Approved by Thomas Fuss

Name           | Role                      | Signature
-------------- | --------------            | --------------
Thomas Fuss    | Head of IT and Operations | Thomas Fuss

 

Document Version Control:

Version        | Date           | Author         | Description
-------------- | -------------- | -------------- | --------------
0.1            | 22-Sep-2015    | Nitin Deshmukh | Initial Draft
0.2            | 19-Jan-2016    | Nitin Deshmukh | Added new APIs
 

This document describes workflows and APIs for integrating Payconiq as a white label payment solution. Partner will receive an Identifier and Access Token from Payconiq. It’s mandatory to provide partner’s ID as a URL path parameter and Access token as Authorisation header for all APIs listed in this document unless it is stated explicitly. APIs are available on Testing and Production environment and are accessible using following URLs -


Testing [https://dev.payconiq.com](https://dev.payconiq.com)  
Production [https://api.payconiq.com](https://api.payconiq.com)  
Please note that the SSL certificate is self signed for Testing URL. This will be changed in future.  


# Work Flows

This Chapter covers workflows for Payconiq Partner Integration. APIs used in workflows are covered in subsequent chapter.

## Partner Administration
 
* Request Partner Details
* Register RSA Public Key

## Customer On-boarding & Administration

* Enrolment of new customer with provided mandate reference and sign date  
* Request Mandate and create Mandate in case mandate reference and sign date is not provided at the time of enrolment of customer  
* Getting Customer details(Prerequisite: Either mandate reference and sign date should be provided during customer enrolment or mandate is created explicitly using Create Mandate API)  
* Updating customer Phone/Email
    
## Customer Account Administration

* Requesting 1 ct transaction for Account Verification  
* Verifying Account using code received as a description of 1 ct transaction

## Initiating Transaction

* Initiating Transaction from Customer to Partner
* Replay of failed SDD transaction

## Transaction Reporting

* Requesting a Transaction details
* Requesting Paginated List of transactions with filters and date range

# API Resources

This Chapter covers all API Resources for Payconiq Partner Integration. APIs used in workflows are covered in subsequent chapter.

## Get Partner Information
This API returns information related to Partner.

### Request
Resource URL:

HTTP Method    | URL
-------------- | --------------
GET            | /v1/partners/{id}

Request Parameters


### Response
Response Body
 
 status        | element        | MediaType        
-------------- | -------------- | --------------
 200           |    partner     | Application Json 
 403/401       | errorResponse  | Application Json 

Possible Error Response

Following two error response are applicable for all API calls hence not listed in subsequent APIs.


 status               | Error Code            | level                 | Values                | Message
--------------------- | --------------------- | --------------------- | --------------------- | ---------------------
 401                  |    R918               | Error                 |                       | Token not found.
 403                  | R981                  | Error                 | "GET", "resource url" | Security exception : you cannot call GET on partners/560012d5c25c4168ce03dd54
 500                  | R999                  | Error                 |                       | Internal Server Error
 
### Sample Request Response
 
> Sample Request:

```shell
GET https://dev.payconiq.com/v1/partners/560012d5c25c4168ce03dd54
Authorization: 81a6bd5d-f02d-4278-aad6-d42a64d3f45d
```

> Sample Response:

```json
{
"_id":"",
"firstName":"",
"lastName":"",
"phone":"",
"email":"",
"address":{
"no":"",
"street":"",
"postalCode":"",
"city":"",
"country":"NLD"
},
"bankAccounts":[
{
"name":"ING",
"status":"VALIDATED",
"IBAN":""
}
],
"creationDate":1442845397482,
"companyName":"",
"type":"partner",
"setting":{
"enableSMSVerification":false
}
}
```
> Sample Error Response:

```json
{
"level":"ERROR",
"code":"R981",
"message":"Security exception : you cannot call GET on partners
560012d5c25c4168ce03dd546",
"values":[
"GET",
"partners/560012d5c25c4168ce03dd546"
]
}
```

## Update Partner RSA Public Key

This API registers RSA Public Key for signature verification. Once key is registered, subsequent
key updates requires signature generated using Base64 Encode (SHA 256 Hash(new key, old
private key). Signature will be verified by API using old public key before registering new key.

### Request

Resource URL

HTTP Method    | URL
-------------- | --------------
PUT            | /v1/partners/{id}/key

Request Parameters

Name           | Description            | Type           | Mandatory      | Default
-------------- | --------------         | -------------- | -------------- | --------------
 Authorization | Authentication token   | Head           | Yes            | 
 id            | ID of partner          | Path           | Yes            | 
 
Request Body

element        | MediaType
-------------- | --------------
key            | Application Json

### Response

Response

status         | element          | MediaType
-------------- | --------------   | --------------
200            | Application Json | 
403/401/400    | errorResponse    | Application Json

Possible Error Response

status           | Error Code            | level           | Values         | Message
--------------   | --------------        | --------------  | -------------- | --------------
 400             | FIELD_IS_REQUIRED     | Error           | value          | Field 'value' is mandatory
 403             | INVALID_SIGNATURE     | Error           |                | Provided signature is invalid
 
### Sample Request Response

> Sample Request

```shell
PUT https://dev.payconiq.com/v1/partners/560012d5c25c4168ce03dd54/key
Authorization: 81a6bd5d-f02d-4278-aad6-d42a64d3f45d
Content-Type: application/json
{
"value":"<Base64 Encode(RSA Public Key)>",
"signature":"<Base64 Encode(SHA 256 Hash(New RSA Public Key, Old private key))>"
}

Response : 200 OK
```
## Register new Partner Customer

This API registers new Partner Customer with provided SDD Mandate reference and sign date. If
mandate reference and sign date is not provided then API generates Mandate Reference. In this
case Mandate needs to be created explicitly using create mandate API to complete customer
registration.

### Request

Resource URL

HTTP Method        | URL
-------------- | --------------
POST            | /v1/partners/{id}/customers

Request Parameters

Name           | Description            | Type           | Mandatory      | Default
-------------- | --------------         | -------------- | -------------- | --------------
 Authorization | Authentication token   | Head           | Yes            | 
 id            | ID of partner          | Path           | Yes            | 
 
 Request Body
 
 element        | MediaType
 -------------- | --------------
 customer       | Application Json
 
### Response
 
 Response
 
 status         | element          | MediaType
 -------------- | --------------   | --------------
 201            |                  | 
 403/401/400    | errorResponse    | Application Json

Response Header

status         | Header          | Value        
-------------- | --------------  | --------------
 201           |    Location     | Location(URL with ID of customer) of registered Customer 
 
 
 Possible Error Response
 
 status           | Error Code            | level           | Values                 | Message
 --------------   | --------------        | --------------  | --------------         | --------------
  400             | FIELD_IS_REQUIRED     | Error           | "firstName"            | Field 'firstName' is mandatory
  400             | FIELD_IS_REQUIRED     | Error           | "lastName"             | Field 'lastName' is mandatory
  400             | FIELD_IS_REQUIRED     | Error           | "phoneNumber"          | Field 'phoneNumber' is mandatory
  400             | FIELD_IS_REQUIRED     | Error           | "addres"               | Field 'addres' is mandatory
  400             | FIELD_IS_REQUIRED     | Error           | "no"                   | Field 'no' is mandatory
  400             | FIELD_IS_REQUIRED     | Error           | "street"               | Field 'street' is mandatory
  400             | FIELD_IS_REQUIRED     | Error           | "postalCode"           | Field 'postalCode' is mandatory
  400             | FIELD_IS_REQUIRED     | Error           | "city"                 | Field 'postalCode' is mandatory
  400             | FIELD_IS_REQUIRED     | Error           | "IBAN"                 | Field 'city' is mandatory
  400             | FIELD_IS_REQUIRED     | Error           | "name"                 | Field 'IBAN' is mandatory
  400             | FIELD_IS_REQUIRED     | Error           | "contry"               | Field 'contry' is mandatory
  400             | FIELD_IS_REQUIRED     | Error           | "bankAccounts"         | Field 'bankAccounts' is mandatory
  400             | FIELD_IS_REQUIRED     | Error           | "mandateReference"     | Field 'mandateReference' is mandatory
  400             | FIELD_IS_REQUIRED     | Error           | "mandateSignDate""     | Field 'mandateSignDate' is mandatory
  400             | R111                  | Error           |                        | More than one Bank Account are specified
  400             | R106                  | Error           |                        | Field 'email' is not correct
  400             | R107                  | Error           |                        | Field 'phoneNumber' is not correct
  400             | R108                  | Error           |                        | Field 'firstName' is not correct
  400             | R109                  | Error           |                        | Field 'lastName' is not correct
  400             | R122                  | Error           |                        | Field 'IBAN' is not valid
  
  
### Sample Request Response

Sample Request: 

POST https://dev.payconiq.com/v1/partners/560012d5c25c4168ce03dd54/customers
Authorization: 81a6bd5d-f02d-4278-aad6-d42a64d3f45d
Content-Type: application/json

```json
{
"firstName":"",
"lastName":"",
"phone":"",
"email":"",
"address":{
"street":"",
"no":"",
"postalCode":"",
"city":"",
"country":"NLD"
},
"bankAccounts":[
{
"IBAN":"",
"name":"ING",
“mandateReference":""
"mandateSignDate":
}
]
}
```

Response

201
Location: https://dev.payconiq.com/v1/partners/560012d5c25c4168ce03dd54/customers/<_id>


## Update Partner Customer email/phone

This API updates Partner Customer’s email and phone.

### Request

Resource URL:

 HTTP Method     | URL
 -------------- | --------------
 PATCH          | /v1/partners/{id}/customers/{customer-id}
 
Request Parameters: 
 
 Name              | Description            | Type            | Mandatory      | Default
  --------------   | --------------         | --------------  | -------------- | --------------
   Authorization   | Authentication token   | Head            | Yes            | 
   id              | ID of partner          | Paths           | Yes            |  
   customer-id     | ID of partner customer | Paths           | Yes            | 
   
Request Body: 

element         | MediaType
 -------------- | --------------
 customer       | Application Json
 
### Response

Response:

 status        | element        | MediaType        
-------------- | -------------- | --------------
 200           |                | 
 403/401       | errorResponse  | 
 
Possible Error Response
 
  status               | Error Code            | level                 | Values                     | Message
 --------------------- | --------------------- | --------------------- | ---------------------      | ---------------------
  400                  | R106                  | Error                 |                            | Field 'email' is not correct
  400                  | R107                  | Error                 |                            | Field 'phoneNumber' is not correct
  400                  | ENTITY_NOT_FOUND      | Error                 | "Partner Customer", "<id>" | The Partner Customer with the id '<id>' does not exist
  
### Sample Request Response

Sample Request: 

PATCH https://dev.payconiq.com/v1/partners/560012d5c25c4168ce03dd54/customers/
560013e4c25c4168ce03dd56
Authorization: 81a6bd5d-f02d-4278-aad6-d42a64d3f45d
Content-Type: application/json

```json

{
"phone":"",
"email":"",
}
```

Response : 200 OK

## Get Partner Customer Details

This API returns information of Partner Customer.

### Request

Resource URL:

HTTP Method     | URL
 -------------- | --------------
 GET            | /v1/partners/{id}/customers/{customer-id}
 
Request Parameters:

Name           | Description            | Type           | Mandatory      | Default
-------------- | --------------         | -------------- | -------------- | --------------
 Authorization | Authentication token   | Head           | Yes            | 
 id            | ID of partner          | Path           | Yes            | 
 customer-id   | ID of Customer         | Path           | Yes            |  
 
### Response

Response:


 status          | element         | MediaType        
--------------   | --------------  | --------------
 200             | partnerCustomer | 
 400/401/403/404 | errorResponse   | Application Json
 
 Possible Error Response:
 
  status                | Error Code            | level                 | Values                     | Message
  --------------------- | --------------------- | --------------------- | ---------------------      | ---------------------
   404                  | ENTITY_NOT_FOUND      | Error                 | "Partner Customer"         | The Partner Customer with the id '<id>' does not exist
   400                  | R508                  | Error                 |                            | Mandate does not exist for given user and account 
   
### Sample Request Response

Sample Request

GET https://dev.payconiq.com/v1/partners/560012d5c25c4168ce03dd54/customers/
560013e4c25c4168ce03dd56
Authorization: 81a6bd5d-f02d-4278-aad6-d42a64d3f45d

Sample Response:

```json
{
“_id":"",
"firstName":"",
"lastName":"",
"address":{
"no":"",
"street":"",
"postalCode":"",
"city":"",
"country":""
},
"bankAccounts":[
{
"name":"ING",
"status":"NOT_VALIDATED",
"IBAN":""
"mandateReference":"",
“mandateSignDate”:
}
]
}
```

## Get Partner Customer Account Mandate

This API returns Mandate Text, Reference, Debtor and Creditor details.

### Request

Resource URL:

HTTP Method     | URL
 -------------- | --------------
 GET            | /v1/partners/{id}/customers/{customer-id}/bankAccounts/{iban}/mandate

Request Parameters:

Name             | Description                                                          | Type           | Mandatory      | Default
--------------   | --------------                                                       | -------------- | -------------- | --------------
 Authorization   | Authentication token                                                 | Head           | Yes            | 
 Accept-Language | Accept Language header for Madnate text and reason. Either en or nl  | Head           | No             | nl
 id              | ID of partner                                                        | Path           | Yes            |  
 customer-id     | ID of Customer                                                       | Path           | Yes            | 
 iban            | IBAN of Customer Account                                             | Path           | Yes            | 
 
### Response

Response:


 status          | element         | MediaType        
--------------   | --------------  | --------------
 200             | mandate         | 
 400/401/403/404 | errorResponse   | Application Json


Possible Error Response

 status               | Error Code            | level                 | Values                    | Message
--------------------- | --------------------- | --------------------- | ---------------------     | ---------------------
 401                  | ENTITY_NOT_FOUND      | Error                 | "Partner Customer", "<id>"| The Partner Customer with the id '<id>' does not exist
 400                  | R122                  | Error                 |                           | Field 'IBAN' is not valid
 
### Sample Request Response

Sample Request

GET https://dev.payconiq.com/v1/partners/560012d5c25c4168ce03dd54/customers/
560013e4c25c4168ce03dd56/bankAccounts/DE21200600000143457000/mandate
Authorization: 81a6bd5d-f02d-4278-aad6-d42a64d3f45d
Accept-Language: en

Sample Response:

```json

{
"reference":"<mandate reference>",
"debtorName":"",
"debtorIBAN":"",
"debtorAddress":"",
"creditorAddress":"",
"signDate":1442845668291,
"text":"By clicking \"agree\" below, you authorise payconiq to send recurrent SEPA direct debit
collection instructions to your bank to debit your account and you authorise your bank to debit
your account in accordance with the instructions from payconiq.\n\nAs part of your rights, you are
entitled to a refund from your bank under the terms and conditions of your agreement with your
bank. A refund must be claimed within 8 weeks starting from the date on which your account was
debited. Your rights are explained in a statement that you can obtain from your bank.",
"reason":"<Partner Name> Payments"
}
```

## Create Partner Customer Account Mandate

This API creates Mandate of Partner Customer account. This API is only applicable if Mandate
Reference and Sign Date was not provided while enrolling customer.

### Request

Resource URL:

HTTP Method     | URL
 -------------- | --------------
 POST           | /v1/partners/{id}/customers/{customer-id}/bankAccounts/{iban}/mandate

Request Parameters:

Name             | Description                                                          | Type           | Mandatory      | Default
--------------   | --------------                                                       | -------------- | -------------- | --------------
 Authorization   | Authentication token                                                 | Head           | Yes            | 
 id              | ID of partner                                                        | Path           | Yes            |  
 customer-id     | ID of Customer                                                       | Path           | Yes            | 
 iban            | IBAN of Customer Account                                             | Path           | Yes            | 
 
### Response

Response:


 status          | element         | MediaType        
--------------   | --------------  | --------------
 200             |                 | 
 400/401/403/404 | errorResponse   | Application Json


Possible Error Response

 status               | Error Code            | level                 | Values                    | Message
--------------------- | --------------------- | --------------------- | ---------------------     | ---------------------
 401                  | ENTITY_NOT_FOUND      | Error                 | "Partner Customer", "<id>"| The Partner Customer with the id '<id>' does not exist
 400                  | R122                  | Error                 |                           | Field 'IBAN' is not valid

### Sample Request Response

Sample Request

POST https://dev.payconiq.com/v1/partners/560012d5c25c4168ce03dd54/customers/
560013e4c25c4168ce03dd56/bankAccounts/DE21200600000143457000/mandate
Authorization: 81a6bd5d-f02d-4278-aad6-d42a64d3f45d

Response: 201
Location: https://dev.payconiq.com/v1/partners/560012d5c25c4168ce03dd54/customers/<_id>/
bankAccounts/DE21200600000143457000/mandate

## Cancel Partner Customer Account Mandate

This API cancels Mandate of Partner Customer account.

### Request

Resource URL:

HTTP Method     | URL
 -------------- | --------------
 POST           | /v1/partners/{id}/customers/{customer-id}/bankAccounts/{iban}/mandate/cancel

Request Parameters:

Name             | Description                                                          | Type           | Mandatory      | Default
--------------   | --------------                                                       | -------------- | -------------- | --------------
 Authorization   | Authentication token                                                 | Head           | Yes            | 
 id              | ID of partner                                                        | Path           | Yes            |  
 customer-id     | ID of Customer                                                       | Path           | Yes            | 
 iban            | IBAN of Customer Account                                             | Path           | Yes            | 
 
### Response

Response:


 status          | element         | MediaType        
--------------   | --------------  | --------------
 201             | mandate         | 
 400/401/403/404 | errorResponse   | Application Json


Possible Error Response

 status               | Error Code            | level                 | Values                    | Message
--------------------- | --------------------- | --------------------- | ---------------------     | ---------------------
 401                  | ENTITY_NOT_FOUND      | Error                 | "Partner Customer", "<id>"| The Partner Customer with the id '<id>' does not exist
 400                  | R122                  | Error                 |                           | Field 'IBAN' is not valid

### Sample Request Response

Sample Request:

POST https://dev.payconiq.com/v1/partners/560012d5c25c4168ce03dd54/customers/
560013e4c25c4168ce03dd56/bankAccounts/DE21200600000143457000/mandate/cancel
Authorization: 81a6bd5d-f02d-4278-aad6-d42a64d3f45d

Response 200 OK

## Trigger 1ct transaction for account verification

This API initiates 1ct transaction with verification code as a description of transaction for Partner
Customer Account Verification.

### Request

Resource URL:

HTTP Method     | URL
 -------------- | --------------
 POST           | /v1/partners/{id}/customers/{customer-id}/bankAccounts/{iban}/sendVerificationCode

Request Parameters:

Name             | Description                                                          | Type           | Mandatory      | Default
--------------   | --------------                                                       | -------------- | -------------- | --------------
 Authorization   | Authentication token                                                 | Head           | Yes            | 
 id              | ID of partner                                                        | Path           | Yes            |  
 customer-id     | ID of Customer                                                       | Path           | Yes            | 
 iban            | IBAN of Customer Account                                             | Path           | Yes            | 
 
### Response

Response:


 status          | element         | MediaType        
--------------   | --------------  | --------------
 200             | mandate         | 
 400/401/403/404 | errorResponse   | Application Json


Possible Error Response

 status               | Error Code            | level                 | Values                    | Message
--------------------- | --------------------- | --------------------- | ---------------------     | ---------------------
 401                  | ENTITY_NOT_FOUND      | Error                 | "Partner Customer", "<id>"| The Partner Customer with the id '<id>' does not exist
 400                  | R122                  | Error                 |                           | Field 'IBAN' is not valid

Sample Request Response

POST https://dev.payconiq.com/v1/partners/560012d5c25c4168ce03dd54/customers/
560013e4c25c4168ce03dd56/bankAccounts/DE21200600000143457000/sendVerificationCode
Authorization: 81a6bd5d-f02d-4278-aad6-d42a64d3f45d

Response: 200 OK

## Validate Account

This API validates Partner Customer account against verification code that was sent as a
description of 1ct transaction.

### Request

Resource URL:

HTTP Method     | URL
 -------------- | --------------
 POST           | /v1/partners/{id}/customers/{customer-id}/bankAccounts/{iban}/validate

Request Parameters:

Name             | Description                                                          | Type           | Mandatory      | Default
--------------   | --------------                                                       | -------------- | -------------- | --------------
 Authorization   | Authentication token                                                 | Head           | Yes            | 
 id              | ID of partner                                                        | Path           | Yes            |  
 customer-id     | ID of Customer                                                       | Path           | Yes            | 
 iban            | IBAN of Customer Account                                             | Path           | Yes            | 
 
### Response

Response:


 status          | element         | MediaType        
--------------   | --------------  | --------------
 200             | mandate         | 
 400/401/403/404 | errorResponse   | Application Json


Possible Error Response

 status               | Error Code            | level                 | Values                    | Message
--------------------- | --------------------- | --------------------- | ---------------------     | ---------------------
 401                  | ENTITY_NOT_FOUND      | Error                 | "Partner Customer", "<id>"| The Partner Customer with the id '<id>' does not exist
 400                  | R122                  | Error                 |                           | Field 'IBAN' is not valid
 400                  | R930                  | Error                 |                           | Too many attempts with wrong authentication code
 400                  | R919                  | Error                 |                           | Bank Account verification code not found
 400                  | R214                  | Error                 |                           | Verification code is not correct
 
 ### Sample Request Response
 
 Sample Request:

POST https://dev.payconiq.com/v1/partners/560012d5c25c4168ce03dd54/customers/
560013e4c25c4168ce03dd56/bankAccounts/DE21200600000143457000/validate
Authorization: 81a6bd5d-f02d-4278-aad6-d42a64d3f45d\
Content-Type: application/json
{
"verificationCode":"<code>"
}

Response: 200 OK


### Initiate Transaction

This API initiates transaction from Customer to Partner. Customer IBAN is optional. If it’s not
specified then API derives it from primary account associated with Customer ID. Partner IBAN is
derived from Partner ID.
Before initiating transaction it’s mandatory for Partner to register RSA Public key since it is used for
transaction signature verification. Partner needs to generate transaction signature (Base64
Encode(SHA256WithRSA Encrypt(concat(partnerId, originId, optional origin Iban, amount,
currency), Private key)) using associated Private key. Please note that if customer IBAN is
specified in transaction body then it’s mandatory to include it in signature. API verifies signature
using registered RSA Public Key. Please note that the algorithm used for generating signature
should be SHA256WithRSA.

### Request

Resource URL:

HTTP Method     | URL
 -------------- | --------------
 POST           | /v1/partners/{id}/transactions

Request Parameters:

Name             | Description                                                          | Type           | Mandatory      | Default
--------------   | --------------                                                       | -------------- | -------------- | --------------
 Authorization   | Authentication token                                                 | Head           | Yes            | 
 id              | ID of partner                                                        | Path           | Yes            |  

 Request Body
 
 element        | MediaType
 -------------- | --------------
 transaction    | Application Json
 
### Response

Response:


 status          | element         | MediaType        
--------------   | --------------  | --------------
 201             | mandate         | 
 400/401/403/404 | errorResponse   | Application Json


Possible Error Response

 status               | Error Code               | level                 | Values                               | Message
--------------------- | ---------------------    | --------------------- | ---------------------                | ---------------------
 401                  | ENTITY_NOT_FOUND         | Error                 | "Partner Customer", "<id>"           | The Partner Customer with the id '<id>' does not exist
 400                  | FIELD_IS_REQUIRED        | Error                 | “originId/amount/currency/signature“ | Field 'originId/amount/currency/signature' is mandatory
 400                  | R013                     | Error                 |                                      | Transaction 'originId' is not known by the system
 400                  | R003                     | Error                 |                                      | Transaction field 'amount' is null or not a decimal number
 400                  | R017                     | Error                 |                                      | Transaction amount has to be greater than 0
 400                  | R002                     | Error                 |                                      | Transaction field ''currency'' can only be EUR for now
 400                  | AUTHENTICATION_NOT_FOUND | Error                 | 'Public Key'                         | Public Key' authentication does not exist. You have to add one.
 400                  | INVALID_SIGNATURE        | Error                 |                                      | Provided signature is invalid
 
 ### Sample Request Response
 
 Sample Request:

POST https://dev.payconiq.com/v1/partners/560012d5c25c4168ce03dd54/transactions
Authorization: 81a6bd5d-f02d-4278-aad6-d42a64d3f45d\
Content-Type: application/json
{
"originId":"<customer id>",
"originIban":"<customer IBAN>",
"signature":"Base64 Encode(SHA256WithRSA Encrypt(concat(partnerId, originId, optional
originIban, amount, currency), private key)",
"amount":<amount in cents>,
"currency":"EUR"
}

Response:
201
Location: https://dev.payconiq.com/v1/partners/560012d5c25c4168ce03dd54/transactions/<id>

## Get Transaction

This API returns information of transaction.

### Request

Resource URL:

HTTP Method     | URL
 -------------- | --------------
 POST           | /v1/partners/{id}/transactions/{transaction-id}

Request Parameters:

Name             | Description                                                          | Type           | Mandatory      | Default
--------------   | --------------                                                       | -------------- | -------------- | --------------
 Authorization   | Authentication token                                                 | Head           | Yes            | 
 id              | ID of partner                                                        | Path           | Yes            |  
 customer-id     | ID of Customer                                                       | Path           | Yes            | 
 
### Response

Response:


 status          | element         | MediaType        
--------------   | --------------  | --------------
 200             | mandate         | 
 400/401/403/404 | errorResponse   | Application Json


Possible Error Response

 status               | Error Code            | level                 | Values                             | Message
--------------------- | --------------------- | --------------------- | ---------------------              | ---------------------
 404                  | ENTITY_NOT_FOUND      | Error                 | "Transaction", "<treanscation-id"> | The Transaction with the id ‘<transaction-id>' does not exist
 R019                 | FIELD_IS_REQUIRED     | Error                 |                                    | Invalid transaction ID : {0}
 R020                 | R013                  | Error                 |                                    | Missing transaction ID


Possible SDD Failed Reasons

In case SDD processing of transactions fails then sddFailedReason field of transaction will contain
one of following error codes

 Code           | Description
 -------------- | --------------
 MD06           | Return of funds requested by end customer
 AM04           | Insufficient funds
 FOCR           | Return following a cancellation request
 AC01           | Incorrect account number
 AC04           | Account closed
 AC06           | Blocked account
 AC13           | Debtor account type is missing or invalid
 BE01           | Debtor’s name does not match account holder name
 BE01           | Debtor’s name does not match account holder name
 MD07           | End customer deceased
 RR02           | Specification of debtor's name and/or address needed for reasons of regulatory requirements is insufficient or missing
 AG01           | Transaction forbidden
 RR01           | Regulatory reason
 RR04           | Regulatory reason
 MS02           | Reason has not been specified by end customer
 MS03           | Reason has not been specified by Agent
 SL01           | Due to specific services offered by debtor agent
 AG02           | Invalid bank operation code
 AM05           | Duplication
 BE05           | Creditor identification incorrect
 CNOR           | Creditor bank is not registered under this BIC in CSM
 DNOR           | Debtor bank is not registered under this BIC in CSM
 FF01           | Operations/transaction code incorrect, invalid file format
 FF05           | Direct Debit Type Incorrect
 MD01           | No mandate
 MD02           | Missing or Incorrect Mandatory Mandate Information
 RC01           | Bank identifier incorrect
 RR03           | Missing creditor name or address
 AC03           | Invalid Creditor Account Number
 AGNT           | Incorrect agent
 BE06           | Unknown End Customer
 CUTA           | Cancelation requested because an investigation has been requested and no remediation is possible
 ED01           | Correspondent Bank Not Possible
 ED05           | Settlement Failed
 XT79           | Debtor agent not allowed to receive DD
 XT80           | Credit agent not allowed to send DD
 CUST           | Requested By Customer
 EMVL           | EMV Liability Shift
 NARR           | Narrative
 SL02           | Specific Service offered by Creditor Agent
 SVNR           | Service Not Rendered
 XT87           | R-Msg not following same DP route / sending DP not identical to instructing / instructed agent of the original transaction
 AM01           | ZeroAmount
 AM02           | Not Allowed Amount
 AM06           | Too Low Amount
 AM07           | Blocked Amount
 AM09           | Wrong Amount
 AM10           | Invalid Control Sum
 ARDT           | Already Returned Transaction
 CURR           | Incorrect Currency
 DT01           | Invalid Date
 ED03           | Balance Info Request
 MD04           | Invalid Fileformat
 MD05           | Creditor/creditor's agent should not have collected the DD
 NOAS           | No Answer From Customer
 NOOR           | No Original Transaction Received
 PY01           | Correspondent bank not possible
 RC07           | Invalid Creditor BIC Identifier
 RF01           | Not Unique Transaction Reference
 TECH           | Technical problems resulting in erroneous SDD's
 XT78           | Check on compensation amount in refunds failed
 
### Sample Request Response

Sample Request:

GET https://dev.payconiq.com/v1/partners/560012d5c25c4168ce03dd54/transactions/
56051d8cc25c4173d7987290
Authorization: 81a6bd5d-f02d-4278-aad6-d42a64d3f45d

Response:

{
"_id":"56051d8cc25c4173d7987290",
"originId":"5601709bc25c416c9ede5ca7",
"originIBAN":"",
"targetIBAN":"",
"targetId":"560012d5c25c4168ce03dd54",
"amount":"1000",
"currency":"EUR",
"originUser":{
"_id":"5601709bc25c416c9ede5ca7",
"firstName":"",
"lastName":"",
"address":{
"no":"",
"street":"",
"postalCode":"",
"city":"Amsterdam",
"country":"NLD"
},
"bankAccounts":[
{
"name":"",
"status":"",
"IBAN":""
}
]
},
"targetUser":{
"_id":"560012d5c25c4168ce03dd54",
"firstName":"",
"lastName":"",
"phone":"",
"email":"",
"address":{
"no":"",
"street":"",
"postalCode":"",
"city":"",
"country":"NLD"
},
"bankAccounts":[
{
"name":"ING",
"status":"",
"IBAN":""
}
],
“creationDate":1442845397482,
"companyName":"",
"type":"partner"
},
"creationDate":1443175820938, "status":"SUCCEEDED"
"sddProcessDate":1443175820938,
"sddFailedReason": <please refer to Possible SDD failed reason section above>
}

## Get Transactions

This API returns paginated list of transactions based on from-to, customer ID and transaction
status filters.

### Request

Resource URL:

HTTP Method     | URL
 -------------- | --------------
 POST           | /v1/partners/{id}/transactions

Request Parameters:

Name             | Description                                                          | Type           | Mandatory      | Default
--------------   | --------------                                                       | -------------- | -------------- | --------------
 Authorization   | Authentication token                                                 | Head           | Yes            | 
 id              | ID of partner                                                        | Path           | Yes            |  
 limit           | Number of transactions per page                                      | Query String   | Optional       | 25
 offset          | Page number                                                          | Query String   | Optional       | 1
 
Request Body
 
 element          | MediaType
 --------------   | --------------
 transactionQuery | Application Json

 
### Response

Response:


 status          | element         | MediaType        
--------------   | --------------  | --------------
 200             | mandate         | 
 400/401/403/404 | errorResponse   | Application Json

### Sample Request Response

Sample Request

GET https://dev.payconiq.com/v1/partners/560012d5c25c4168ce03dd54/transactions?
limit=25&offset=1
Authorization: 81a6bd5d-f02d-4278-aad6-d42a64d3f45d
Content-Type: application/json
{
“from":<unix timestamp>,
"to":<unix timestamp>,
"status":"SUCCEEDED/FAILED",
"sddStatus":"EXCEPTION/SUBMITED/REJECTED_OSR/REJECTED_CAMT"
}

Response:

[{
"_id":"56051d8cc25c4173d7987290",
"originId":"5601709bc25c416c9ede5ca7",
"originIBAN":"",
"targetIBAN":"",
"targetId":"560012d5c25c4168ce03dd54",
"amount":"1000",
"currency":"EUR",
"originUser":{
"_id":"5601709bc25c416c9ede5ca7",
"firstName":"",
"lastName":"",
"address":{
"no":"",
"street":"",
"postalCode":"",
"city":"Amsterdam",
"country":"NLD"
},
"bankAccounts":[
{
"name":"",
"status":"",
"IBAN":""
}
]
},
"targetUser":{
"_id":"560012d5c25c4168ce03dd54",
"firstName":"",
"lastName":"",
"phone":"",
"email":"",
"address":{
"no":"",
"street":"",
"postalCode":"",
"city":"",
"country":"NLD"
},
"bankAccounts":[
{
"name":"ING",
"status":"",
"IBAN":""
}
],
"creationDate":1442845397482,
“companyName":"", "type":"partner"
},
“creationDate":1443175820938, “status":"SUCCEEDED"
"sddProcessDate":1443175820938,
"sddFailedReason": <please refer to Possible SDD failed reason section above>
} ]


## Replay Failed SDD

This API replays failed SDD transaction.

### Request

Resource URL:

HTTP Method     | URL
 -------------- | --------------
 POST           | /v1/partners/{id}/transactions/{transaction-id}/replaySDD

Request Parameters:

Name             | Description                                                          | Type           | Mandatory      | Default
--------------   | --------------                                                       | -------------- | -------------- | --------------
 Authorization   | Authentication token                                                 | Head           | Yes            | 
 id              | ID of partner                                                        | Path           | Yes            |  
 transaction-id  | ID of Transaction                                                    | Path           | Yes            |  
 
### Response

Response:


 status          | element         | MediaType        
--------------   | --------------  | --------------
 200             | mandate         | 
 400/401/403/404 | errorResponse   | Application Json


Possible Error Response

 status               | Error Code            | level                 | Values                    | Message
--------------------- | --------------------- | --------------------- | ---------------------     | ---------------------
 404                  | ENTITY_NOT_FOUND      | Error                 | "Transaction", "<id>"     | The Transaction with the id '<id>' does not exist
 400                  | R165                  | Error                 |                           | Transaction replay is not allowed
 400                  | R165                  | Error                 |                           | Transaction replay is not allowed for <sdd-status> status

### Sample Request Response

Sample Request:

POST https://dev.payconiq.com/v1/partners/560012d5c25c4168ce03dd54/transactions/
560013e4c25c4168ce03dd56/replaySDD
Authorization: 81a6bd5d-f02d-4278-aad6-d42a64d3f45d

Response: 200 OK