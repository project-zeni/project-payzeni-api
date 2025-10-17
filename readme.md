
# Payzeni API Guide

## Introduction

We ​require **all merchants**​ to complete a full test cycle in our sandbox environment using the information provided below before any production information can be set up and released. Once testing has been completed, contact us to enable your live account.

Throughout this document, placeholders are used to represent variable elements.  These placeholders are enclosed with “{ }”.  Following are the explanations for these:

1. `{host}` – this refers to the host portion of a URL
   * contact support team for url details
2. `{merchantId}`
   * A merchant ID and API key will be issued to you upon creation of your account
   * Your merchant ID and API key in the test environment will be different from that of your production account
3. `{hashed token}`
   * The hashing requirements are explained in a separate document from support team.
  


## Submit Payment Request 

Payment API calls will require a `{hashed token}` to be passed as part of the request header. Talk to the support team to check how to create the hash.


| Description                         | Submit a payment request |                                  
|:------------------------------- |:------------------------------------------------------- |
| HTTP Method                         | POST                                                   |
| Endpoint URL                        | `https://{host}/mxapi/{merchantId}/paymentrequest/submit` |
| Request Headers                     | `Content-Type: application/json;`              <br> `X-payzeni-apiauthz: {hashed token}` |                                             |
| Request Body      | In JSON format. Attributes as described below.             |

**_Request Body Attributes:_**
|Attribute Name	| Required ? |Type	|Description
|:----------------------------------- |:--- |:--- |:---------------------------------------------------------- |
|orderId| Y	| String	| Your own transaction/order identifier |
|amount	| Y | Numeric	| The requested amount to be paid by the customer |
|customerEmail | Y	|String |	 The customer’s email address |
|customerName	| Y | String	| The customer’s full name |
|customerPhone | N |	String | The customer’s phone number |
|paymentDesc |	Y | String	| A description of the payment|
|callbackUrl	| N | String|	Your endpoint URL for receiving callbacks when the status of the transaction changes. If not specified, no callback will be made. |
|dupCheck | N |	boolean	| When set to true, the system will reject a request if a transaction with the same orderId already exists. Defaults to false if not specified.|
|sendEmail | N	| boolean	| When set to true, Interac will send an email to the customer with the payment link. Defaults to true if not specified. |

**_Sample Request_**

```
{
"customerEmail": "test@somedomain.com",
"customerName": "Test Test",
"orderId": "xyz123",
"amount": 10.00,
"paymentDesc": "Test interac e-transfer request"
}
```

**_Response_**

If the request fails authorization of validation, you will get an HTTP Status of 400 (Bad Request). The response body will contain an error code and an error message in a JSON-formatted text. For example:

```
{
"msgCode": "E2012" ,
"msgText": "Customer Name is required"
}
```

If the request succeeds, you will get an HTTP Status of 200. The response body will contain the details of the transaction in JSON-formatted text with the following attributes:

**_Response Attributes:_**

|Attribute Name	|Type	|Description
|:----------------------------------- |:--- |:---------------------------------------------------------- |
|txnId	| Numeric	| Payzeni’s transaction identifier |
|orderId	|String	|Your transaction/order identifier as specified in the request|
|createDate	|String|	The date/time the transaction was created|
|status	|String	|The current status of the transaction. Possible values: <br>`PENDING` <br>	`COMPLETED` <br>	`CANCELLED` <br>	`REJECTED` <br>	`TIMEDOUT` <br>	`SYSERR`
|statusDate |	String|	The date/time the current status was set
|requestedAmount |	Numeric	|The requested amount
|processedAmount	|Numeric|	The amount processed/fulfilled. This will be zero until the customer fulfills the payment request
|currency |	String	|This will always be CAD|
|customerEmail	| String	| The customer’s email as specified in the request |
|customerName	| String	| The customer’s full name as specified in the request|
|customerPhone	| String	| The customer’s phone if specified in the request. Otherwise it will be null|
|paymentLink	| String	| This is the URL that will take the customer to the Interac payment gateway page where this will select the financial institution from which they will fulfill the request. You may display this link as the end-state of your payment flow or automatically redirect to it. If the “sendEmail” attribute in the request is set to true, the customer would receive an email containing the same link.|
|message	| String	|This is a message relating to the status of the transaction| 

**_Sample Response_**
```

    {
    "txnId": 2584206965130,
    "createDate": "2025-03-10 23:31:36 UTC",
    "status": "PENDING",
    "statusDate": "2025-03-10 23:31:36 UTC",
    "orderId": "xyz123",
    "paymentLink": "https://test.payzeni.com/mockpaylinkpage.html?rID=FKWWV4RQIN",
    "requestedAmount": 10.00,
    "processedAmount": 0,
    "currency": "CAD",
    "customerEmail": "test@somedomain.com",
    "customerName": "Test Test",
    "customerPhone": null,
    "message": "Mock ETransfer Request Sent"
    }
```

**Callback**

When the status of a transaction changes, the system will send a callback if a “callbackUrl” was supplied in the request. This will be in the form of an HTTP Post with Content-Type of “application/json”. The body would contain the details of the transaction in JSON-formatted text. The attributes will be exactly the same as those of the response attributes described above. Your endpoint that will receive this callback need only return an HTTP 200. No content is necessary. Any content in your response will be ignored.



## Query Payment Request


This API call will require a **{hashed token}** to be passed as part of the request header as specified below. Talk to the support team on how to generate the hash using the following parameters.

* `merchantId`

* `txnId`

* `apiKey`

**_REQUEST CALL_**

Set the hash value in the request header with a header name of “X-payzeni-apiauthz”.

|Description |Query a payment request |
|:---|:---|
| HTTP Method |POST|
| Endpoint URL | `https://{host}/mxapi/{merchantId}/paymentrequest/query` |
| Request Headers | `Content-Type: application/json;` <br> `X-payzeni-apiauthz: {hashed token}`
|Request Body |In JSON format. Attributes as described below. |

**_Request Body Attributes:_**


 | **Attribute Name** | **Required?** | **Type** |**Description**|
|--|---|--|--|
| txnId| Y |Numeric |The Payzeni transaction identifier|

**_Sample Request_**
```
{
"txnId": 2584206965130
}
```
**Response**

If the request succeeds, you will get an HTTP Status of 200. The response body will contain the details of the transaction in JSON-formatted text. The attributes will be exactly the same as those of the response attributes described above.
