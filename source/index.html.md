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

# Test GitHub connection

Partner API Specification Approved by Thomas Fuss

Name | Role | Signature
-------------- | -------------- | --------------
Thomas Fuss | Head of IT and Operations | Thomas Fuss

 

Document Version Control:

Version | Date | Author | Description
-------------- | -------------- | -------------- | --------------
0.1 | 22-Sep-2015 | Nitin Deshmukh | Initial Draft
0.2 | 19-Jan-2016 | Nitin Deshmukh | Added new APIs
 

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

HTTP Method | URL
-------------- | --------------
GET | /v1/partners/{id}

|  Name   |  Description   |  Type   |   Mandatory  |    Default |
| --- | --- | --- | --- | --- |
|   Authorization  | Authentication token    |   Head  |   Yes  |     |
|  id   | ID of partner    |  Path   |   Yes  |     |

### Response

Response Body
