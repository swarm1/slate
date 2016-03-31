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


 status        | Error Code     | level          | Values         | Message
-------------- | -------------- | -------------- | -------------- | --------------
 401           |    R918     | AError |  | Token not found.
 403           | R981  | Error | "GET", "resource url" | Security exception : you cannot call GET on partners/560012d5c25c4168ce03dd54
 500           | R999  | Error | | Internal Server Error
 
### Sample Request Response
 
Sample Request:

GET https://dev.payconiq.com/v1/partners/560012d5c25c4168ce03dd54
Authorization: 81a6bd5d-f02d-4278-aad6-d42a64d3f45d

Sample Response

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

Sample Error Response

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
-------------- | -------------- | -------------- | -------------- | --------------
 Authorization | Authentication token   | Head           | Yes            | 
 id            | ID of partner          | Path           | Yes            | 
 
Request Body

element        | MediaType
-------------- | --------------
key            | Application Json

### Response

Response

status         | element          | MediaType
-------------- | -------------- | --------------
200            | Application Json | 
403/401/400    | errorResponse | Application Json

Possible Error Response

status           | Error Code            | level           | Values      | Message
-------------- | -------------- | -------------- | -------------- | --------------
 400 | FIELD_IS_REQUIRED  | Error           | value            | Field 'value' is mandatory
 403 | INVALID_SIGNATURE  | Error           |            | Provided signature is invalid
 
### Sample Request Response

Sample Request

PUT https://dev.payconiq.com/v1/partners/560012d5c25c4168ce03dd54/key
Authorization: 81a6bd5d-f02d-4278-aad6-d42a64d3f45d
Content-Type: application/json
{
"value":"<Base64 Encode(RSA Public Key)>",
"signature":"<Base64 Encode(SHA 256 Hash(New RSA Public Key, Old private key))>"
}

Response : 200 OK

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
-------------- | -------------- | -------------- | -------------- | --------------
 Authorization | Authentication token   | Head           | Yes            | 
 id            | ID of partner          | Path           | Yes            | 
 
 Request Body
 
 element     | MediaType
 -------------- | --------------
 customer            | Application Json
 
### Response
 
 Response
 
 status         | element          | MediaType
 -------------- | -------------- | --------------
 201            |  | 
 403/401/400    | errorResponse | Application Json

Response Header

status        | Header        | Value        
-------------- | -------------- | --------------
 201           |    Location     | Location(URL with ID of customer) of registered Customer 
 
 
 Possible Error Response
 
 status           | Error Code            | level           | Values      | Message
 -------------- | -------------- | -------------- | -------------- | --------------
  400 | FIELD_IS_REQUIRED  | Error           | "firstName"            | Field 'firstName' is mandatory
  400 | FIELD_IS_REQUIRED  | Error           | "lastName"             | Field 'lastName' is mandatory
  400 | FIELD_IS_REQUIRED  | Error           | "phoneNumber"          | Field 'phoneNumber' is mandatory