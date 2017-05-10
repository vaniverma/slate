---
title: API Reference

language_tabs:

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>


search: true
---


# About OLP

> ![warehouse image][warehouse]

OLP (Open Logistics Platform) is a multi-tenant Courier Logistics Platform. It is a completely automated solution to manage the picking of a shipment from a seller or warehouse to its delivery to the end customer’s doorstep. It is an independent system which will be used by any ecommerce client.

There are various sub-systems communicating with each other to accomplish various operations involved in managing the OLP.

The features available for an ecommerce client are as highlighted below:


- __Real-time Create shipment requests for select Logistics provider and fetch tracking numbers__. This means, providing shipment data for requesting either a shipment pick up from a serviceable location or to deliver a shipment to a serviceable location. The location can be a seller/ warehouse or an end customer. This translates to the following scenarios:
  - Scheduling shipment pickup from ecommerce client and deliver to end customer.
  - Scheduling reverse-pickup requests from end customers and deliver to eCommerce client.
  - Scheduling shipment pickup from sellers directly and deliver to end customers.
  - Scheduling reverse pickup requests from end customers and deliver to any location like a QC Centre or Seller.
- __A variety of Value Added Services can be provided at the time of a shipment pickup or delivery. These include__
  - Capturing electronic proof of pickup and delivery at the time of handover,
  - Capturing photographic proof of pickup and delivery and Capturing the delivery, 
  - Pickup location coordinates i.e., lat long of the place of pickup and delivery.
- __The system is capable of real time syncs of all the above mentioned tasks to the commerce client.__
- __The system offers to provide Best-in-class Customer Experience. It can provide system triggered__
   - Arriving Soon for Delivery/Pick up SMS notification,
   - Arrived at your door-step SMS notification,
   - Real time Shipment cancellation request (at any point in time of the shipment journey, you can trigger to cancel the delivery or pickup. The details of this will be covered in detail in the cancellation api section),
   - Slotted deliveries and pick ups (To be attempted in a given range of date and time),
   - Reschedule Shipment Deliveries,
   - Option to Leave with neighbour.


# Overview

This guide describes the integration with OLP. It has been designed to assist the developers to understand the API structure and definition. We also intend to explain how to use these APIs.

To get started with using OLP APIs:

1. Create a User account with our system.
2. Check out a few tutorials on how to use the API: cURL, Postman.

Before getting into the explanation of the APIs, let us learn about what is a shipment and what do we mean by a shipment journey.

## Shipment

> ![Describing shipment][shipment]

If in a shipment journey, it is picked from a seller or warehouse - we call it its first mile journey. It is then brought into the Courier Provider’s facility and follows inter-facility journey (between hubs and DC), and then is sent out for delivery to the end-customer - we call it last mile journey.

## What can the APIs do?
You can use the OLP APIs to create shipments within our system and manage its journey from its pick up from a warehouse or seller, across hubs until it finally reaches its destination - the customer’s address and vice-versa.

## Shipment Journey

> ![Shipment Journey][shipment_journey]

A look at the flow of events:

 - Create a Shipment in OLP - this internally creates journey for the
   shipment.
 - You can reschedule the pickup or delivery of the shipment or both.
 - You can cancel the shipment at pickup - This ensures that the shipment is not picked up from the seller or warehouse.
 - You can also cancel the shipment before it is out for delivery(from the Distribution Centre) - This ensures that the shipment is not delivered to the destination address.
 - If the shipment is already out for delivery, then it can not be cancelled.

## Rest APIs
The base URL for all requests to the OLP API is:
> https://host:port/v{i}/shipments

Our APIs are REST-based. This means:

- It makes use of standard HTTP verbs like GET, POST, DELETE, PUT and PATCH.
- The API uses standard HTTP error responses to describe errors. 
- Token-based Authentication is used in accessing all the OLP APIs.
- The data in the above mentioned methods should be encoded as standard application/json.

## Rate Limiting

To maintain performance and availability across diverse base of clients, we have to maintain the number of hits within the limits of capacity of our APIs.

Rate limiting of the API is primarily on a per-account group basis. That is, every account group will be allowed a limited number of API hits, for say, a window of 15 minutes.

__Implementation:__

`X-RateLimit-Limit-Second`: The rate limit ceiling for that given endpoint in 1 second, for example, if the API is hit at 11:00:00, then the defined number of API hits are allowed till 11:00:01.
`X-RateLimit-Remaining-Second`: The number of requests left for the 1 second window. 
`X-RateLimit-Limit-Minute`: The rate limit ceiling for that given endpoint in 1 minute. 
`X-RateLimit-Remaining-Minute`: The number of requests left for the 1 minute window.
`X-RateLimit-Limit-Hour`: The rate limit ceiling for that given endpoint in 1 hour. 
`X-RateLimit-Remaining-Hour`: The number of requests left for the 1 hour window.

Please see the example below:
`X-RateLimit-Limit-Second`: 100 
`X-RateLimit-Remaining-Second`: 25 
`X-RateLimit-Limit-Minute`: 4000 
`X-RateLimit-Remaining-Minute`: 2000

>Please Note: When an application exceeds the rate limit for a given API endpoint, the API will return a HTTP 429 “Too Many Requests” response code.

# Client Account On-boarding in OLP

For any new client on-boarding on to the OLP platform, the below inputs are needed as a onetime setup. These should be clarified with the account POC as part of requirements and operational process discussion.

## Account creation in OLP
An account code is uniquely used to identify the client account and its shipments in OLP system. The account code needs to be created in OLP using whose reference the shipments will be created to differentiate between multiple client’s shipments in OLP. Contact the OLP product point of contact to have an account created in staging before integration testing. This same activity needs to be done in production as well once before go-live.

## Create shipment in OLP
The shipment can be created in two ways in OLP. The preferred and readily available option is 1 as given below. Option 2 is not readily available and would need close discussion with client before finalising the format.

 - _Option 1_ : The shipments are created in OLP system via the API
   integration call real time. In response to the API call, the shipment
   tracking number will be provided. Ideally clients make this API call
   at the time of courier hard allocation at their end i.e., when the
   client has decided on allocating the said shipment to Vulcan OLP
   courier. The API documentation provided gives all the details of the
   API.
 - _Option 2_ : The second option to create shipments in OLP is by way of
   shipment data upload via import. This is available to internal Vulcan
   operations team only. The client shares the shipment soft data
   allocated to Vulcan courier via mail or other method. The Vulcan team
   in turn uploads the data to create shipment in system before in scan
   of shipments. There is a defined shipment soft data format in which
   the client is expected to share the data.

> __Cons__: Serviceability check errors possible. Also, whenever courier expands serviceability, there is a delay in manual sync up leading to under-utilisation of courier services

__Sort Code__
Sort code is the value to be printed on the shipment label along with the tracking number. Sort code helps the courier to sort the shipment efficiently. Sort code is again maintained in 2 ways as given below. Option 1 is preferred way of using sort code.

 - _Option 1_ : Sort code is maintained by the client as part of the pin
   code serviceability master maintained for a courier. At the time of
   pack slip print, client refers to this offline master and prints the
   sort code in the shipment label.
 - _Option 2_ : Along with API integration for create shipment, tracking
   number and sort code will be provided in response to the create
   shipment call. This is currently not available and hence option 1 is
   what suggested to clients.


## External Client/Customer Facilities

Client’s warehouse/fulfilment Centres/QC Centres etc. are pre-created in OLP system to facilitate bulk pickup and delivery of shipments. There is a format for sharing this data and to be created as a one-time setup before go-live. The required fields for an external customer facility are as below:

1. Facility Code
2. Facility Name
3. Address
a. Name
b. Address line 1
c. Address line 2
d. City
e. State
f. Pin code
g. Phone
h. Email

If the external customer facility is setup once in OLP, at the time of create shipment, the external customer code could be passed in place of the pickup address code.

## Pickup Shipment Request

Post shipment creation, there is a pickup request sent to OLP for when the shipments are ready for pickup by the courier. This helps the courier plan for the pickup as and when the client is ready with the shipments. Usually clients call the pickup API call when a group of shipments are manifested together.

 - _Option 1_ : A pickup request usually contains a list of tracking
   numbers and a date time slot for the pickup.
 - _Option 2_ : In case pickup API is not called, then the operations team
   work out a time schedule between them to schedule a regular pickup at
   a fixed routine.

## Tracking Status Updates

The tracking events are pushed to the client systems. The available statuses and its meaning are available as part of the API documentation. Client’s could map the OLP status events to their internal system statuses. The tracking events can only be pushed currently. PULL based status updates are not available. As a backup, Vulcan can also provide the .csv file download of shipment and its statuses on an agreed frequency. The PUSH based status events are sent via web hooks of which details are available in the API documentation.

## Shipment Attempt Count for delivery

The number of attempts for delivery before a shipment is returned to the client. This is configurable as per the client and the contract terms between client & courier. It’s usually 3 attempts after which the client has two options:

 - _Option 1_: The shipment can be put into a wait state where the neither
   further delivery attempt would be made nor the shipment would be
   returned. The client has the option to contact his/her customer and
   check about re-delivery. Accordingly, client could initiate a return
   or re-attempt for delivery by either calling cancel request or
   reschedule request real time respectively. More details on the cancel
   and reschedule APIs in further sections.
 - _Option 2_ : Post 3 unsuccessful attempts, shipments are automatically
   marked for return. The client will not have an option attempt
   delivery again. This case, the shipments will always be marked
   returned.


## Shipment Cancellation Request

Shipment delivery can be cancelled real time by calling cancellation API request at any point in time. Upon successful cancellation of delivery as per eligible cancellation statuses, the client would receive a successful response in the API call. If the shipment is already out for delivery, then the cancellation request is rejected. All other statuses, the cancellation is accepted.

There is also an alternative way for sending the cancellation requests to Vulcan operations by mail where against a list of AWB numbers, cancel request can be shared. The downside of this offline approach is that the cancellation request cannot be guaranteed i.e., by the time the sheet is uploaded to the system the shipment might be delivered to customers. Hence API integration for cancellation is much preferred.

## Shipment Reschedule Request

Shipment reschedule request can be initiated real time via API calls. Alternatively, offline reschedule request via file upload is also supported. A list of tracking numbers with reschedule instruction, reschedule date time, reschedule address in case of incorrect customer addresses are supported. Further details in the API documentation.

Below is the quick check list of the items discussed above in the document.

> ![Shipment reschedule table 1][shipment_reschedule_1]
> ![Shipment reschedule table 2][shipment_reschedule_2]

# Authentication
The OLP APIs require authentication using JWT token which ensures that the user accesses a protected route or resource. You must provide a valid username and password.

## API: Login

| Method        |              URL           | 
| ------------- |:--------------------------:| 
| POST          | `https://<authentication host>/oauth/token?grant_type=password&client_id=my-trusted-client&poolType=ACCOUNT_GROUP&poolCode=SNAPDEAL` | 


Every time a user logs in with valid credentials, an access token is generated, which is then sent in the authentication headers of every API hit and validated.

__Header__ - `Content-Type: application/x-www-form-urlencoded`
__Request Body__ - `username=XXXX&password=YYYY`

__Response__:
__Content-Type:__ `application/json`
The response body for successful requests is a JSON object with the following structure:

```json
{ 
"access_token": "eyJhbGciOiJIUzI1NiJ9.eyJleHBpcmF0aW9uIjoxNDg3MjUzMDAyOTI2LCJpc3N1ZWRBdCI6MTQ4NzI0MzAwMjkyNiwiaXNzdWVyIjoiT2xwQXV0aCIsInR5cGUiOiJhY2Nlc3NfdG9rZW4iLCJpc3N1ZWRGb3IiOiJha 3NoYXkuYWdhcndhbEB1bmljb21tZXJjZS5jb20iLCJ0ZW5hbnRDb2RlIjoidnVsY2FuIiwib3RoZXJDbGFpbXMiOnt9fQ.Vyzn0jftlyjrK5xCDic5jsMU81elxpR4s6_vFCcCAb8", 
"token_type": "bearer", 
"refresh_token": "eyJhbGciOiJIUzI1NiJ9.eyJleHBpcmF0aW9uIjoxNDg3NzQzMDAyOTI2LCJpc3N1ZWRBdCI6MTQ4NzI0MzAwMjkyNiwiaXNzdWVyIjoiT2xwQXV0aCIsInR5cGUiOiJyZWZyZXNoX3Rva2VuIiwiaXNzdWVkRm9yIjoiYWtzaGF5LmFnYXJ3YWxAdW5pY29tbWVyY2UuY29tIiwidGVuYW50Q29kZSI6InZ1bGNhbiIsImNsaWVudF9pZCI6Im15LXRydXN0ZWQtY2xpZW50IiwidXNlcl9uYW1lIjoiYWtzaGF5LmFnYXJ3YWxAdW5pY29tbWVyY2UuY29tIiwiYXRpIjoiMTE4NGMwODctZDI3MC00NGY5LWEyNmEtYzBmM2VlMGMwZT I3Iiwic2NvcGUiOlsidHJ1c3QiLCJ3cml0ZSIsInJlYWQiXSwib3RoZXJDbGFpbXMiOnt9fQ.MzWv_nJQ r9Dk pj9YVvL9f2uvpxC19cUXt8IUqznfs",
"expires_in": 9999,  
}
```

| Fields        | Data Type     | Description  | Sample Value  |
| ------------- |:-------------:| -----------: | -------------:|
| Access Token  | String        | The unique token generated for offering access to a specific resource for a time period        | eyJhbGciOiJIUzI1NiJ9.eyJleHBpcmF0aW9uIjoxNDg3MjUzMDAyOTI2LCJpc3N1ZWRBdCI6MTQ4NzI0MzAwMjkyNiwiaXNzdWVyIjoiT2xwQXV0aCIsInR5cGUiOiJhY2Nlc3NfdG9rZW4iLCJpc3N1ZWRGb3IiOiJha3NoYXkuYWdhcndhbEB1bmljb21tZXJjZS5jb20iLCJ0ZW5hbnRDb2RlIjoidnVsY2FuIiwib3RoZXJDbGFpbXMiOnt9fQ.Vyzn0jftlyjrK5xCDic5jsMU81elxpR4s6_vFCcCAb8| 
| Token Type    | String        |   This represents how an access token is generated at an authorisation server       |  Bearer |
| Refresh Token | String        |    Allows the refreshing of the previously generated Access token        |    eyJhbGciOiJIUzI1NiJ9.eyJleHBpcmF0aW9uIjoxNDg3NzQzMDAyOTI2LCJpc3N1ZWRBdCI6MTQ4NzI0MzAwMjkyNiwiaXNzdWVyIjoiT2xwQXV0aCIsInR5cGUiOiJyZWZyZXNoX3Rva2VuIiwiaXNzdWVkRm9yIjoiYWtzaGF5LmFnYXJ3YWxAdW5pY29tbWVyY2UuY29tIiwidGVuYW50Q29kZSI6InZ1bGNhbiIsImNsaWVudF9pZCI6Im15LXRydXN0ZWQtY2xpZW50IiwidXNlcl9uYW1lIjoiYWtzaGF5LmFnYXJ3YWxAdW5pY29tbWVyY2UuY29tIiwiYXRpIjoiMTE4NGMwODctZDI3MC00NGY5LWEyNmEtYzBmM2VlMGMwZTI3Iiwic2NvcGUiOlsidHJ1c3QiLCJ3cml0ZSIsInJlYWQiXSwib3RoZXJDbGFpbXMiOnt9fQ.MzWv_nJQr9Dkpj9YVvL9f2uvpxC19cUXt8IUqznfs |
| Expires In    | Long          |   Time period for which an Access token is valid (in seconds)     |    9999   |



Description of fields in the above JSON are provided below:


| Status Code   | Response Body | 
| ------------- |:-------------:| 
| 200 OK        | See the JSON structure above. | 
| 401           | Unauthorised Access      | 
| 403           | Token Expired      | 


__Errors:__

If your access token expires then make use of the following API to generate a new access token

| Method   | URL | 
| ------------- |:-------------:| 
| POST       | https://{authentication host}/oauth/token? grant_type=refresh_token&client_id=my-trusted- client | 




__Header__ - `Content-Type: application/x-www-form-urlencoded`
__Request Body__ - `refresh_token=XXXX`

__Response__:
__Content-Type:__ `application/json`
The response body for successful requests is a JSON object with the following structure:

```json
{ 
"access_token": "eyJhbGciOiJIUzI1NiJ9.eyJleHBpcmF0aW9uIjoxNDg3MjUzMDAyOTI2LCJpc3N1ZWRBdCI6MTQ4NzI0MzAwMjkyNiwiaXNzdWVyIjoiT2xwQXV0aCIsInR5cGUiOiJhY2Nlc3NfdG9rZW4iLCJpc3N1ZWRGb3IiOiJha 3NoYXkuYWdhcndhbEB1bmljb21tZXJjZS5jb20iLCJ0ZW5hbnRDb2RlIjoidnVsY2FuIiwib3RoZXJDbGFpbXMiOnt9fQ.Vyzn0jftlyjrK5xCDic5jsMU81elxpR4s6_vFCcCAb8", 
"token_type": "bearer", 
"refresh_token": "eyJhbGciOiJIUzI1NiJ9.eyJleHBpcmF0aW9uIjoxNDg3NzQzMDAyOTI2LCJpc3N1ZWRBdCI6MTQ4NzI0MzAwMjkyNiwiaXNzdWVyIjoiT2xwQXV0aCIsInR5cGUiOiJyZWZyZXNoX3Rva2VuIiwiaXNzdWVkRm9yIjoiYWtzaGF5LmFnYXJ3YWxAdW5pY29tbWVyY2UuY29tIiwidGVuYW50Q29kZSI6InZ1bGNhbiIsImNsaWVudF9pZCI6Im15LXRydXN0ZWQtY2xpZW50IiwidXNlcl9uYW1lIjoiYWtzaGF5LmFnYXJ3YWxAdW5pY29tbWVyY2UuY29tIiwiYXRpIjoiMTE4NGMwODctZDI3MC00NGY5LWEyNmEtYzBmM2VlMGMwZT I3Iiwic2NvcGUiOlsidHJ1c3QiLCJ3cml0ZSIsInJlYWQiXSwib3RoZXJDbGFpbXMiOnt9fQ.MzWv_nJQ r9Dk pj9YVvL9f2uvpxC19cUXt8IUqznfs",
"expires_in": 9999,  
}
```

Whenever the user wants to access a protected route or resource, the user agent should send the JWT, typically in the Authorization header using the Bearer schema. This is a stateless authentication mechanism as the user state is never saved in server memory. The server's protected routes will check for a valid JWT in the Authorization header, and if it's present, the user will be allowed to access protected resources. As JWTs are self- contained, all the necessary information is there, reducing the need to query the database multiple times.

This allows you to fully rely on data APIs that are stateless and even make requests to downstream services.

[Image explaining JWT flow]

# APIs

There are a number of Objects defined in the system, which are used in the APIs. These are defined as below:

## Shipment Object
The Shipment object used in APIs to create, update and get shipments has the following JSON structure:

__Request Body:__

> Shipment Object Request Body:

```json
{
  "lengthInMm": 100,
  "widthInMm": 100,
  "heightInMm": 100,
  "weightInGram": 100,
  "collectibleAmount": 323.0,
  "monetaryValue": 335.0,
  "handoverMode": "PICKUP",
  "fulfillmentTat": "1468649548633",
  "referenceNumber": "SLP1251560804",
  "shipmentType": "DELIVERY",
  "currencyCode": "INR",
  "paymentMethodCode": "COD",
  "destinationAddress": {
    "name": "Sanjeev",
    "addressLine1": "20/B,New Railway Colony,Knox Road,Chotta Baghara,Prayag,Allahabad",
    "addressLine2": "",
    "city": "Allahabad",
    "state": "Uttar Pradesh",
    "countryCode": "IN",
    "pincode": "211002",
    "phone": "9621836972"
  },
  "originAddress": {
    "name": "Sasikala",
    "addressLine1": "Standard Silk Mills,Lal Darwaja,Vasta Devdi Road,Near Ayurvedic College Hostel,Sardar Nagar Road",
    "addressLine2": "",
    "city": "Surat"
  },
  "state": "Gujarat",
  "countryCode": "IN",
  "pincode": "395003",
  "phone": "9712013074",
  "shipmentItems": [{
    "identifier": "Deepak Blue Pure Georgette Saree",
    "name": "Deepak Blue Pure Georgette Saree",
    "shipperAddress": {
      "name": "Sasikala",
      "addressLine1": "Standard Silk Mills,Lal Darwaja,Vasta Devdi Road,Near Ayurvedic College Hostel,Sardar Nagar Road",
      "addressLine2": "",
      "city": "Surat",
      "state": "Gujarat",
      "countryCode": "IN",
      "pincode": "395003",
      "phone": "9712013074"
    }
  }],
  "soldToAddress": {
    "name": "Sanjeev",
    "addressLine1": "20/B,New Railway Colony,Knox Road,Chotta Baghara,Prayag,Allahabad",
    "addressLine2": "",
    "city": "Allahabad",
    "state": "Uttar Pradesh",
    "countryCode": "IN",
    "pincode": "211002",
    "phone": "9621836972"
  }
}
```

Description of fields in the above JSON are provided below:

|Fields  | Data Type  | Description  | Sample Value | Mandatory/ Optional/ Not Required
| ------- |:-------------:|:--------------|:--------------------:|:-----------------| 
| lengthInMm | Integer |The length of the Shipping Package (in mm) |100 |No Default Value | M |
| widthInMm  | Integer  | The width of the Shipping Package (in mm) |100 | No Default Value |M  |
| heightInMm | Integer  |The height of the Shipping Package (in mm) |100 |No Default Value | M |
|weightInGram |Number|The weight of the Shipping Package (in gm)|1000 No Default Value|M|
|monetaryValue|Number|The total monetary value of package (in defined currency code)|5000 No Default Value|M|
|collectibleAmount|Number|The total amount to be collected from the customer, in case of COD (It can contain Product MRP, Delivery charges, Gift wrapping charges,whichever|applicable)|7000 No Default Value|M if PaymentMedtho d is COD|
|handoverMode|Enum|"Drop" if you have to handover the shipment at destination or "Pickup" if you have to pick up a shipment from the destination.|{“DROP”/ “PICKUP”} No Default Value|M|
|accountCode|String|The Client Code, for example, in case of Vulcan, Snapdeal is an account, so mention the Code for Snapdeal here.|Snapdeal No Default Value|M|
|fulfillmentTAT|String|The turn-around time (in ms) within which the shipment has to be delivered to the customer.|1468649548633 No Default Value|O|
|referenceNumber|String|The code or number given by the account (eg. Snapdeal) to identify a shipment.|SLP1251560804 No Default Value|M|
|shipmentType|Enum|"Delivery" if you have to handover the shipment at destination or "Reverse Pickup" if you have to pick up a shipment from the destination.|{“DELIVERY”/ “REVERSEPIC KUP”} No Default Value|M|
|currencyCode|String|The code of the currency in which the payment is to be made by the customer. It represents all the money related fields.|INR, No Default Value|M|
|paymentMethod Code|Enum|The mode of payment of the customer (COD/ Prepaid).|{“COD”/”PREP AID”} No Default Value|M|
|destinationAddre ssId|String|The Destination Facility Code/ External Customer Code. For End Customer’s address, this field is not applicable|“BAMLPC" No Default Value|O|
|destinationAddress|Object:Address|The Address of the end customer|```{"name": "Rajshree Jaiswal","address Line1": "O/33, Fatehpur 1st lane Garden Reach kolkata-700024","addressLine2": "","city": "Kolkata","state": "West Bengal","country Code": "+91","pincode": null,"phone": "7278643841","position": [88.298449516, 22.527621236]}``` No default Value|M only if destinationAddressId is not provided
|originAddressId|String|The Origin facility address code.|“BAMLPC" ,No Default Value|O|
|originAddress|Object:Address|The Address of the point of origin (fetched from Location Master)|{"name": "OS- SUR-VASTA-VL CENTER","addr essLine1": "Standard Silk Mills,Lal Darwaja,Vasta Devdi Road,Near Ayurvedic College Hostel,Sardar Nagar Road","addressLi ne2": "","city": "Surat","state": "Gujarat","countr yCode": "IN","pincode": null,"phone": "9712013074","p osition": [72.8349875, 21.2074527]} No Default Value|M only if originAddressId is not provided|
|shipmentItem s|Array [ShipmentIte m]|Items present in a shipment|[“set of cups”,”set of spoons”,”set of plates”] No Default Value|O|
|trackingNumber|String|The AWB number assigned to the shipment while dispatching with the courier provider.|AXBNR567843 No Default Value|M|
|soldToAddressId|String|The Destination Facility Code/ External Customer Code. For End Customer’s address, this field is not applicable|“BAMLPC" No Default Value|O|
|soldToAddress|Object: Address|Customer Billing Address (fetched from Location Master).|{“name": "Rajshree Jaiswal","address Line1": "O/33, Fatehpur 1st lane Garden Reach kolkata-700024", "addressLine2": "","city": "Kolkata","state" : "West Bengal","country Code": "+91","pincode": null,"phone": "7278643841","p osition": [88.298449516, 22.527621236] }No default Value|M|
|shipperAddressId|String|The Origin facility address code.|“BAMLPC"  No Default Value|O|
|shipperAddress|Object: Address|The Address of the Seller/ Warehouse|{"name": "OS- SUR-VASTA-VL CENTER","addr essLine1": "Standard Silk Mills,Lal Darwaja,Vasta Devdi Road,Near Ayurvedic College Hostel,Sardar Nagar Road","addressLi ne2": "","city": "Surat","state": "Gujarat","countr yCode": "IN","pincode": null,"phone": "9712013074","p osition": [72.8349875, 21.2074527]} No Default Value|M|
|pickupOptions|Object: Options|The events allowed at pick up and the time slot at which it is scheduled|{"pickupOptions "{"waitForSlots: false,"waitForQc Data": false,"openHand overOption": "OPTIONAL","e scalationReason" : "sdjhfjhsa"}}|O|
|deliveryOptions|Object: Options|The events allowed at delivery and the time slot at which it is scheduled|{"pickupOptions "{"waitForSlots: false,"waitForQc Data": false,"openHand overOption": "OPTIONAL","e scalationReason" : "sdjhfjhsa"}}|O|
> For Object: Options, Please refer to Appendix in this document

__Response Body:__
The object body is similar to the request body, it can however contain many more optional fields. For a reference please check appendix.


## Address Object

The JSON object for an `Address` has the following structure

> Address Object:

```json
 {
   "name": "Aman Chawla",
   "addressLine1": "Standard Mills Compound Lal Darwajavastu Devi Road Near",
   "addressLine2": "Hotel Avon ", 
   "city": "Surat",
   "state": "Gujrat",
   "countryCode": "IN",
   "pincode": "122015",
   "phone": "9878787878",
   "position": [72.8472917, 21.1665957]
 }
```


Description of fields in the above JSON are provided below:

|Fields  | Data Type  | Description  | Sample Value | Mandatory/ Optional/ Not Required
| ------- |:-------------:|:--------------|:--------------------:|:-----------------| 
|name|String|Name of the client or customer|“Aman Chawla” No Default Value|M|
|addressLine 1|String|First line of the physical address of the client facility or customer|“Standard Mills Compound Lal Darwajavastu Devi Road Near" No Default Value|M|
|addressLine 2|String|Second line of the physical address of the client or customer|D-Block, Road number 14, No Default Value|O|
|city|String|City|Surat, No Default Value|M|
|state|String|State|Gujarat, No Default Value|M|
|countryCode|String|Country|India, No Default Value|M|
|pincode|String|The Postal Index Number (PIN) to identify a geographical region within a state in a country|122015, No Default Value|M|
|phone|String|Phone Number   (10 digit mobile number or a landline number with state code)|98XXXXXXXX No Default Value|M|
|position|Array [Double]|Lat Long position of an Address||O|


## Error Object

Restful APIs are not expressive enough to identify he minute details and understand the real reason behind the occurrence of an error. Hence, the system has definition of an error object .

The JSON object for an Error has the following structure:

> JSON object for Error:

```json
{ 
  "code": 103, 
  "message": "No such element exists",  
  "details": "No such shipment exists xyz" 
}
```

|Fields  | Data Type  | Description  | Sample Value |
| ------- |:-------------:|:--------------|:--------------------:|
|code|Integer|The unique code to recognise an error code.|102, 103, 105|
|message|String|Error Heading|No such element exists|
|details|String|Error Details / Summary|No such shipment exists adsfads|


## Time Slot

This object is used for slotted pickup/deliveries where a user demands a shipment pickup/ delivery in a given time slot or range. This means, that the user can tell between which dates and time slots is he available. The system is flexible enough to record different time slots for different days.

The JSON object for a time slot, `TimeSlot` has the following structure:

> JSON Object for `TimeSlot`:

```json
{
  "startTime": "18:30:00",
  "endTime": "20:30:00",
  "startDate": "2017-02-20",
  "endDate": "2017-02-25",
  "type": "DELIVERY"
}
```

Description of fields in the above JSON are provided below:


|Fields  | Data Type  | Description  | Sample Value |
| ------- |:-------------:|:--------------|:--------------------:|
|startTime|String|The start time (in IST) of the time slot/range as defined by the customer.This is in the form HH:MM:SS|18:30:00|
|endTime|String|The end time (in IST) of the time slot/range as defined by the customer.This is in the form HH:MM:SS|20:30:00|
|startDate|String|The start date (in IST) of the time slot/range as defined by the customer.This is in the form YYYY-MM-DD|2017-02-20|
|endDate|String|The end date (in IST) of the time slot/ range as defined by the customer.This is in the form YYYY-MM-DD|2017-02-25|
|type|String|It identifies the type, whether it is a pick up or delivery for which the time slot has been defined. |PICKUP/DELIVERY|


## Pick up Object
This object is used for first mile pick up from sellers. It defines a bunch of shipments to be marked as “Ready for pick up” and the time slot when they should be picked.

The JSON object for an Error has the following structure:

> JSON Object for Error: 

```json
{
  "shipmentCodes": ["JVLSD032214726312", "JVLSD018257468273", "JVLSD011055558884"],
  "timeSlots": [
    {
      "startTime": "18:30:00",
      "endTime": "20:30:00",
      "startDate": "2017-02-20",
      "endDate": "2017-02-25",
      "type": "PICKUP"
    }
  ]
}
```

Description of fields in the above JSON are provided below:

|Fields  | Data Type  | Description  | Sample Value |
| ------- |:-------------:|:--------------|:--------------------:|
|shipmentCodes|Array[String]|List of Shipment code/ AWB number which are ready to be picked from the Seller facility|[“JVLSD032214726312" ,"JVLSD018257468273" ,"JVLSD011055558884" ]|
|timeSlots|Array[TimeSlot]|The object which defines the time slot within which the “ready to pick” shipments can be picked.|[{"startTime": "18:30:00",  "endTime": “20:30:00”,  "startDate": "2017-02-20",  "endDate": "2017-02-25", "type": “PICKUP", }]|


## Event Object
This object identifies the latest action on a shipment during its journey. The JSON object for an `Event` has the following structure:

> JSON Object for `Event`:

```json
 {
   "type": "RECEIVED_AT_HUB",
   "identifier": "788df5f93fc88cf472ef3e84ae1a3356",
   "statusCode": "OUTSCAN_PENDING_FOR_PICKUP",
   "currentFacilityCode": "SURLPC",
   "currentFacilityType": "HUB",
   "address": {
     "city": "Surat",
     "state": "Gujrat",
     "countryCode": "IN"
   },
   "awbNumber": "JVLSD026729430514",
   "eventTimestamp": 1489768118000
 }
```

Description of fields in the above JSON are provided below:

|Fields  | Data Type  | Description  | Sample Value |
| ------- |:-------------:|:--------------|:--------------------:|
|type|String|It identifies the current state of a Shipment|“RECEIVED_AT_HUB”|
|identifier|String|Unique identifier for every event generated for a shipment|“788df5f93fc88cf472 ef3e84ae1a3356”|
|statusCode|String|System defined codes for the current state of a shipment|“OUTSCAN_PENDING_FOR_PICKUP”|
|currentFacilityCode|String|The system defined codes for the Current Facility holding the shipment|"SURLPC"|
|currentFacilityType|String|The type of the facility holding the shipment, e.g., Hub or DC|“HUB"|
|address|Object: Address|Physical address of a facility|{"city": "Surat","state": "Gujrat","countryCod e": "IN"}|
|awbNumber|String|The object which defines the time slot within which the “ready to pick” shipments can be picked.|“JVLSD02672943051 4"|
|eventTimestamp|Datetime|The date and time at which the event was called.|“1489768118000"|


## Shipment Journey Object

This object is used to identify the current state of the journey of a shipment. It highlights the latest event in the shipment journey along with the timeline of events.

The JSON object for a shipment journey has the following structure:

> JSON object for shipment journey: 

```json
{
  "carrierCode": "vulcan",
  "accountCode": "SNAPDEAL",
  "trackingNumber": "CD312662676",
  "referenceNumber": "SHRLMD756902",
  "event": {
    "type": "Event",
    "statusCode": "OUT_FOR_DELIVERY",
    "currentFacilityCode": null,
    "currentFacilityType": null,
    "statusDescription": null,
    "remarks": null,
    "address": {
      "id": "574f21bcbb63da6750ea0523",
      "name": null,
      "addressLine1": "66 E/2 East Babarpur Road Sahadara",
      "addressLine2": null,
      "city": "Delhi",
      "state": "Delhi",
      "countryCode": "IN",
      "pincode": null,
      "phone": null,
      "raw": null,
      "position": [
        77.2090212,
        28.6139391
      ]
    },
    "awbNumber": "CD312662676",
    "child": false,
    "coordinates": null,
    "eventTimestamp": 1486549837000,
    "created": 1486549837000,
    "sequence": 4
  },
  "timeline": [
    {
      "type": "Event",
      "identifier": "389d672b08b8faccf933e8502517009b",
      "statusCode": "PENDING",
      "currentFacilityCode": null,
      "currentFacilityType": null,
      "statusDescription": null,
      "remarks": null,
      "address": null,
      "awbNumber": "CD312662676",
      "child": false,
      "coordinates": null,
      "eventTimestamp": 1486546697000,
      "created": 1486546697000,
      "sequence": 1,
      "updatedBy": null
    },
    {
      "type": "Event",
      "identifier": "389d672b08b8faccf933e8502517009b",
      "statusCode": "RESCHEDULED",
      "currentFacilityCode": null,
      "currentFacilityType": null,
      "statusDescription": null,
      "remarks": null,
      "address": null,
      "awbNumber": "CD312662676",
      "child": false,
      "coordinates": null,
      "eventTimestamp": 1486546697000,
      "created": 1486546697000,
      "sequence": 1,
      "updatedBy": null
    }
  ],
  "availableDocuments": []
}
```

Description of fields in the above JSON are provided below:

|Fields  | Data Type  | Description  | Sample Value | Mandatory/ Optional/ Not Required
| ------- |:-------------:|:--------------|:--------------------:|:-----------------| 
|carrierCode|String|Name of the service provider/ Carrier|“Vulcan”, No Default Value|M|
|accountCode|String|Name of the Client Account availing the service|“Snapdeal", No Default Value|M|
|trackingNumber|String|The Shipment Tracking/ AWB number|“JVLSD029989540212", No Default Value|M|
|referenceNumber|String|The code or number given by the account (eg. Snapdeal) to identify a shipment.|SLP1251560804 No Default Value|M|
|event|Object: Event|The Object which defines the latest event in a shipment’s journey.|{"type": "CREATED","curre ntFacilityCode": null,"currentFacility Type": null,"statusDescripti on": null,"remarks": null,"address": null,"awbNumber": "JVLSD0299895402 12","eventTimestam p": 1489649910000,"seq uence": 1,}|M|
|timeline|Array[Event]|It is a sequence of event(s) occurred in a Shipment’s journey.|Same as above (array of events)|M|
|availableDocume nts|Array[String]|The list of documents sent with the Shipment.|{"event": "DELIVERED","typ e": "IMAGE","url": "https://olp- document.s3.amazon aws.com/ 10c350b4256438928 b25269607a5b78c.jp g"}|O|




Please see the list of some events which are captured in a Shipment’s journey:

CREATED
RECEIVED\_AT\_HUB
IN\_TRANSIT\_BETWEEN\_HUBS
OUT\_FOR\_PICKUP 
ARRIVING\_SOON\_FOR\_PICKUP 
AT\_YOUR\_DOORSTEP\_FOR\_PICKUP 
PICKUP\_CANCELLATION\_REQUESTED 
PICKUP\_RESCHEDULED
PICKED
OUT\_FOR\_DELIVERY 
ARRIVING\_SOON\_FOR\_DELIVERY 
AT\_YOUR\_DOORSTEP\_FOR\_DELIVERY 
DELIVERY\_RESCHEDULED
DELIVERED 
DELIVERY\_CANCELLATION\_REQUESTED 
RETURNED

## A. API: Get Shipment Details

It is called to get the details of a Shipment for the given shipment code.

__Request:__
|Standard HTTP Headers  | Value  |
| --------------------- |:-------------|
|Content-Type | application/json |
|Authorization | &lt;&lt; Auth token &gt;&gt; |

(For Auth token, Please refer to Section 3 of this document)

| Method  | Value  |
| --------------------- |:-------------|
| GET                   | https://host:port/v1/shipments/{code}

__Response:__
|Standard HTTP Headers  | Value  |
| --------------------- |:-------------|
| Content-Type | application/json |
| X-RateLimit-Limit-Second | value |
| X-RateLimit-Remaining-Second | value |
|X-RateLimit-Limit-Minute | value |
|X-RateLimit-Remaining-Minute | value |

The response body for successful requests is a JSON object with the following structure:

> Response body for successful requests in a JSON object:

```json
{ 
... Shipment ...  
}
```

__Errors:__

|Status Code            | Response Body  |
| --------------------- |:-------------|
|200 OK|{...Shipment...}|
|400|Refer Appendix for the detailed list of error codes.|
|403|{"code":1,"details":"User not logged in","message":"User not logged in"}|
|500|{"error": "Something went wrong. Please try again later."}|


## B. API: Create Shipment

It is called to create a shipment in the system.

__Request:__

|Standard HTTP Headers  | Value  |
| --------------------- |:-------------|
|Content-Type | application/json |
|Authorization | &lt;&lt; Auth token &gt;&gt; |
(For Auth token, Please refer to Section 3 of this document)


| Method  | Value  |
| --------------------- |:-------------|
| GET                   | https://host:port/v1/shipments/


> Create Shipment request body:

```json
{ 
... Shipment ...  
}
```

__Response:__

|Standard HTTP Headers  | Value  |
| --------------------- |:-------------|
| Content-Type | application/json |
| X-RateLimit-Limit-Second | value |
| X-RateLimit-Remaining-Second | value |
|X-RateLimit-Limit-Minute | value |
|X-RateLimit-Remaining-Minute | value |


> The response body for successful requests is a JSON object with the following structure:

```json
{ 
... Shipment ...  
}
```


__Errors:__

|Status Code            | Response Body  |
| --------------------- |:-------------|
|200 OK|{...Shipment...}|
|400|Refer Appendix for the detailed list of error codes.|
|401|Access Denied|
|403|{"code":1,"details":"User not logged in","message":"User not logged in"}|
|409|code = 000,No modification: If shipment is already cancelled or return acknowledged|
|500|{"error": "Something went wrong. Please try again later."}|


## C. API: Cancel Shipment

It will cancel the shipment pickup/delivery for the given shipment code. It ensures that the shipment is not processed further and its journey is discarded.

As discussed in the Shipment Journey diagram above, a shipment can be cancelled anytime till it is sent out on a last mile journey (which can be seller/customer pick up or delivery). Once it is sent out for delivery/ pick up, the shipment cannot be cancelled.

__Request:__

|Standard HTTP Headers  | Value  |
| --------------------- |:-------------|
|Content-Type | application/json |
|Authorization | &lt;&lt; Auth token &gt;&gt; |
(For Auth token, Please refer to Section 3 of this document)


| Method  | Value  |
| --------------------- |:-------------|
| GET                   | https://host:port/v1/shipments/{Code}/cancel/


__Response:__

|Standard HTTP Headers  | Value  |
| --------------------- |:-------------|
| Content-Type | application/json |
| X-RateLimit-Limit-Second | value |
| X-RateLimit-Remaining-Second | value |
|X-RateLimit-Limit-Minute | value |
|X-RateLimit-Remaining-Minute | value |


> The response body for successful requests is a JSON object with the following structure:

```json
{ 
... Shipment ...  
}
```


__Errors:__

|Status Code            | Response Body  |
| --------------------- |:-------------|
|200 OK|{...Shipment...}|
|400|Refer Appendix for the detailed list of error codes.|
|401|Access Denied|
|403|{"code":1,"details":"User not logged in","message":"User not logged in"}|
|409|code = 000,No modification: If shipment is already cancelled or return acknowledged|
|500|{"error": "Something went wrong. Please try again later."}|


## D. API: Pickup Shipments
It is called to notify that the given list of shipments are ready to be picked from the Seller at the given time slot.

__Request:__

|Standard HTTP Headers  | Value  |
| --------------------- |:-------------|
|Content-Type | application/json |
|Authorization | &lt;&lt; Auth token &gt;&gt; |
(For Auth token, Please refer to Section 3 of this document)


| Method  | Value  |
| --------------------- |:-------------|
| GET                   | https://host:port/v1/pickups


__Response:__

|Standard HTTP Headers  | Value  |
| --------------------- |:-------------|
| Content-Type | application/json |
| X-RateLimit-Limit-Second | value |
| X-RateLimit-Remaining-Second | value |
|X-RateLimit-Limit-Minute | value |
|X-RateLimit-Remaining-Minute | value |


> The response body for successful requests is a JSON object with the following structure:

```json
{ 
... Pickup ...  
}
```


__Errors:__

|Status Code            | Response Body  |
| --------------------- |:-------------|
|200 OK|{...Pickup...}|
|400|Refer Appendix for the detailed list of error codes.|
|401|Access Denied|
|403|{"code":1,"details":"User not logged in","message":"User not logged in"}|
|409|code = 000,No modification: If shipment is in OUTSCAN_PENDING_FOR_PICKUP|
|500|{"error": "Something went wrong. Please try again later."}|

## E. API: Reschedule Shipment

The API is used to reschedule the delivery/pick up time of the given shipment code at the Origin/ Destination Address. The Origin/ Destination address can be a Seller Warehouse or the Customer.
It allows updating the New time slots, handover activities (if any) and even the Origin/ Destination Address, if provided.

> Please note: Rescheduling a shipment is not allowed once the shipment is out on a last mile journey (pick up from a customer/ seller or delivery).

__Request:__

|Standard HTTP Headers  | Value  |
| --------------------- |:-------------|
|Content-Type | application/json |
|Authorization | &lt;&lt; Auth token &gt;&gt; |
(For Auth token, Please refer to Section 3 of this document)


| Method  | Value  |
| -----------|:-------------|
| GET       | https://host:port/v1/shipments/{Code}/reschedule/


> Request Body: 

```json
{
  "addressType": "DESTINATION",
  "address": {
    "name": "Naresh Kumar",
    "addressLine1": "Plot No.140 pace city sector 37 near hero Honda choke opp. Ausman hospital Gurgaon",
    "addressLine2": "ausman hospital",
    "city": "Gurgaon",
    "state": "Haryana",
    "countryCode": "IN",
    "pincode": "110013",
    "phone": "8860532795"
  },
  "timeSlots": [{
    "startTime": "09:30:04",
    "endTime": "12:30:04",
    "startDate": "2017-03-07",
    "endDate": "2017-03-07",
    "type": "PICKUP"
  }]
}
```
> Please Note: The Address type could be “ORIGIN” or “DESTINATION”.

__Response:__

|Standard HTTP Headers  | Value  |
| --------------------- |:-------------|
| Content-Type | application/json |
| X-RateLimit-Limit-Second | value |
| X-RateLimit-Remaining-Second | value |
|X-RateLimit-Limit-Minute | value |
|X-RateLimit-Remaining-Minute | value |


> The response body for successful requests is a JSON object with the following structure:

```json
{ 
... Shipment ...  
}
```


__Errors:__

|Status Code            | Response Body  |
| --------------------- |:-------------|
|200 OK|{...Shipment...}|
|400|Refer Appendix for the detailed list of error codes.|
|401|Access Denied|
|403|{"code":1,"details":"User not logged in","message":"User not logged in"}|
|409|code = 000,No modification: If shipment is already cancelled or return acknowledged|
|500|{"error": "Something went wrong. Please try again later."}|



> Please note: The system also supports various activities in case of an undelivered or a failed attempt to delivery/pick up.

These activities are configurable and are available as per the client contract. Examples of such activities:

 - Capture digital signature in case of a customer refuses to accept the delivery or pick up.
 - Ask QC questions
 - Capture image of locked door etc.


## E. API: Track Shipment
The API is used to track the journey of a shipment, it highlights the events a shipment has been through in the original sequence.

__Request:__

|Standard HTTP Headers  | Value  |
| --------------------- |:-------------|
|Content-Type | application/json |
|Authorization | &lt;&lt; Auth token &gt;&gt; |
(For Auth token, Please refer to Section 3 of this document)


| Method  | Value  |
| --------------------- |:-------------|
| GET                   | https://host:port/track/{Code}


__Response:__

|Standard HTTP Headers  | Value  |
| --------------------- |:-------------|
| Content-Type | application/json |
| X-RateLimit-Limit-Second | value |
| X-RateLimit-Remaining-Second | value |
|X-RateLimit-Limit-Minute | value |
|X-RateLimit-Remaining-Minute | value |


> The response body for successful requests is a JSON object with the following structure:

```json
{ 
... Shipment Journey...  
}
```


__Errors:__

|Status Code            | Response Body  |
| --------------------- |:-------------|
|200 OK|{...Shipment...}|
|400|Refer Appendix for the detailed list of error codes.|
|401|Access Denied|
|403|{"code":1,"details":"User not logged in","message":"User not logged in"}|
|409|code = 000,No modification: If shipment is already cancelled or return acknowledged|
|500|{"error": "Something went wrong. Please try again later."}|




## Web hooks

A Web hook allows a client to receive real-time updates for certain actions/events on a shipment during its journey to show to its customers. It ensures that the client receives most up-to-date information.

To use web hooks, the client needs to configure his own web application to accept HTTP POST requests from us. These requests will contain JSON objects representing the event that has occurred.

__Supported Event Types__

- ShipmentReschedule
- ShipmentOutforDelivery
- ShipmentArrivingToday
- ShipmentatyourDoorstep
- DeliveryCancellationRequest
- ShipmentDelivered

This list is customisable, as per the contract with the client.


# Appendix

> Response body for a Shipment Object:

```json
{
  "code": "JVLSD018737019454",
  "label": null,
  "identifier": null,
  "waybillNumber": "JVLSD018737019454",
  "lengthInMm": 100,
  "widthInMm": 100,
  "heightInMm": 100,
  "weightInGram": 100,
  "volumetricWeight": 0.2,
  "totalInsuredValue": 0,
  "monetaryValue": 335,
  "type": "SHIPMENT",
  "numberOfPieces": 1,
  "lastFacilityCode": null,
  "currentFacilityCode": "SURLPC",
  "nextFacilityCode": "SURLPC",
  "terminalFacilityCode": "IXDLMD",
  "lastRerouteReason": null,
  "rerouteCount": 0,
  "routeCode": "SUR-RD-BAM-RD-IXD",
  "sortCode": "SUR/IXD",
  "enclosingShipmentCode": null,
  "originAddress": {
    "id": "58352e15bb63da0a699af4c7",
    "name": "Sasikala",
    "addressLine1": "Standard Silk Mills,Lal Darwaja,Vasta Devdi Road,Near Ayurvedic College Hostel,Sardar Nagar Road",
    "addressLine2": "",
    "city": "Surat",
    "state": "Gujarat",
    "countryCode": "IN",
    "pincode": "395003",
    "phone": "9712013074",
    "raw": "Standard Silk Mills,Lal Darwaja,Vasta Devdi Road,Near Ayurvedic College Hostel,Sardar Nagar Road Surat Gujarat 395003 ",
    "position": [72.8349875, 21.2074527]
  },
  "destinationAddress": {
    "id": "583ea3c5bb63da0a699cb2fc",
    "name": "Sanjeev",
    "addressLine1": "20/B,New Railway Colony,Knox Road,Chotta Baghara,Prayag,Allahabad",
    "addressLine2": "",
    "city": "Allahabad",
    "state": "Uttar Pradesh",
    "countryCode": "+91",
    "pincode": "211002",
    "raw": "20/B,New Railway Colony,Knox Road,Chotta Baghara,Prayag,Allahabad Allahabad Uttar Pradesh 211002 ",
    "position": [81.868363, 25.4638175]
  },
  "reason": null,
  "comment": null,
  "updatedBy": null,
  "onHold": false,
  "returnAcknowledged": false,
  "unTraceableStatus": null,
  "unTraceableTimeStamp": null,
  "scanned": false,
  "created": 1487327214411,
  "updated": null,
  "currentTripCode": null,
  "handoverMode": "PICKUP",
  "paymentMethodCode": "COD",
  "shipmentType": "DELIVERY",
  "currencyCode": "INR",
  "referenceNumber": "SLP1251560804",
  "trackingNumber": "JVLSD018737019454",
  "externalStatusCode": "IN_TRANSIT",
  "soldToAddress": {
    "id": "583ea3c5bb63da0a699cb2fc",
    "name": "Sanjeev",
    "addressLine1": "20/B,New Railway Colony,Knox Road,Chotta Baghara,Prayag,Allahabad",
    "city": "Allahabad",
    "state": "Uttar Pradesh",
    "countryCode": "+91",
    "pincode": "211002",
    "phone": "9621836972",
    "raw": "20/B,New Railway Colony,Knox Road,Chotta Baghara,Prayag,Allahabad Allahabad Uttar Pradesh 211002 ",
    "position": [81.868363, 25.4638175]
  },
  "shipperAddress": {
    "id": "58352e15bb63da0a699af4c7",
    "name": "Sasikala",
    "addressLine1": "Standard Silk Mills,Lal Darwaja,Vasta Devdi Road,Near Ayurvedic College Hostel,Sardar Nagar Road",
    "addressLine2": "",
    "city": "Surat",
    "state": "Gujarat",
    "countryCode": "IN",
    "pincode": "395003",
    "phone": "9712013074",
    "raw": "Standard Silk Mills,Lal Darwaja,Vasta Devdi Road,Near Ayurvedic College Hostel,Sardar Nagar Road Surat Gujarat 395003 ",
    "position": [72.8349875, 21.2074527]
  },
  "alternateDestinationAddress": null,
  "returnAddress": {
    "id": "58352e15bb63da0a699af4c7",
    "name": "Sasikala",
    "addressLine1": "Standard Silk Mills,Lal Darwaja,Vasta Devdi Road,Near Ayurvedic College Hostel,Sardar Nagar Road",
    "addressLine2": "",
    "city": "Surat",
    "state": "Gujarat",
    "countryCode": "IN",
    "pincode": "39",
    "phone": "9712013074",
    "raw": "Standard Silk Mills,Lal Darwaja,Vasta Devdi Road,Near Ayurvedic College Hostel,Sardar Nagar Road Surat Gujarat 395003 ",
    "position": [72.8349875, 21.2074527]
  },
  "collectibleAmount": 323,
  "fulfillmentTat": 1468649548633,
  "accountCode": "SNAPDEAL",
  "deliveryDurationInMs": 300000,
  "timeSlots": [],
  "tags": null,
  "shipmentItems": [
    {
      "identifier": "Deepak Blue Pure Georgette Saree",
      "name": "Deepak Blue Pure Georgette Saree",
      "imageUrl": null
    }
  ],
  "shipmentParts": [], 
  "collectibleShipments": [],
  "shipmentTasks": [],
  "returnParentShipmentCode": null,
  "pickupOptions": {
    "slotted": false,
    "doorstepQC": false,
    "openHandoverOption": null
  },
  "deliveryOptions": {
    "slotted": false,
    "doorstepQC": false,
    "openHandoverOption": null
  },
  "returnShipmentCode": null,
  "rescheduleRetryCount": 1,
  "actualRescheduleRetryCount": 0,
  "pickupRescheduleRetryCount": 1,
  "actualPickupRescheduleRetryCount": 0,
  "serviceType": "STANDARD",
  "dg": false,
  "fragile": false
}
```

> Options Body

```json
{
  "pickupOptions": {
    "waitForSlots": false,
    "waitForQcData": false,
    "openHandoverOption": "OPTIONAL",
    "escalationReason": "sdjhfjhsa"
  }
}
```



1. For on-boarding details - The Staging environment is accessible at: vulcan-olpstaging.unicommerce.info For API user name, passwords contact email ID:guhapriya.velu@snapdeal.com
2. For Production issues, contact the emailID: guhapriya.velu@snapdeal.com
3 . Error Mapping with HTTP 400 Error:
__Request:__

|Code | Status  | Description
| --------------------- |:------------| :------------|
|0   |NO\_MODIFICATION    |No modification in the resource|
|101 |NO\_SHIPMENT\_EXISTS |No shipment exists|
|102 |INVALID\_STATUS     |Invalid status|
|103 |NO\_SUCH\_ELEMENT    |No such element exists|
|105 |INVALID\_REQUEST    |Invalid request|
|106 |INVALID\_ADDRESS    |Address is invalid|
|107|DUPLICATE\_SHIPMENT | Shipment already processed|
|109|INVALID\_TIME\_SLOT | slots|
|117|NO\_TRACKING\_NUMBER\_FOUND | All tracking numbers are used|
|119|DUPLICATE\_LANE\_TRIP | Association already exists|
|120|LOST | Lost shipment|
|121|INVALID\_FACILITY | Facility is invalid|
|122|INVALID\_ACTION | Action is invalid|
|125|INVALID\_SHIPMENT\_PIECE\_COUNT | Multipart Shipment- Parts required to be scanned|
|126|PART\_NOT\_ADDED\_TO\_MANIFEST | Missing required part|
|127|ALL\_PARTS\_NOT\_IN\_TRANSIT | All parts not marked in transit|
|129|ALL\_PARTS\_NOT\_RECEIVED | All parts not received|
|130|ALL\_PARTS\_NOT\_OUTSCANNED | All parts not outscanned|
|135|INVALID\_DIMENSIONS | Dimensions are invalid|
|141|FACILITY\_NOT\_FOUND | No facility found|
|142|NOT\_CANCELLABLE |  Not cancellable|
|143|OUT\_OF\_FACILITY | Not in facility|
|144|NO\_ROUTE\_FOUND | No route found|
|145|UNKNOWN\_ERROR |  Unknown error|
|146|FACILITY\_EXISTS | Facility already exists|
|147 |INVALID\_FACILITY\_TYPE | Facility type can only be DC or HUB|
|149 |INVALID\_CLUSTER\_NAME | Invalid cluster code|
|150 |UNEXPECTED\_RESPONSE | Invalid response|
|151 |INVALID\_PATTERN | Invalid pattern|
|152 |INVALID\_CONFIGURATION | Invalid configuration|
|404 |NO\_SORTER\_RESPONSE | Shipment can be sorted|
|154 |DUPLICATE\_COLLECTIBLE\_SHIPMENT | Duplicate collectible shipment|
|155 |BAD\_REQUEST | BAD REQUEST|
|157 |RETURN\_ACKNOWLEDGED | Shipment is return acknowledged|
|158 |SCAN\_SEAL\_CODE | Seal code should be scanned|
|161 |SHIPMENT\_ITEM\_EXISTS | Shipment item already exists|
|701 |FOUND\_ALTERNATE\_ROUTE | Found an alternate route for this shipment|
|800 |Shipment\_NOT\_IN\_FACILITY | Shipment not in current facility|
|50001 |SHIPMENT\_JOURNEY\_NOT\_FOUND | Shipment journey not found|
|429 |TOO\_MANY\_REQUESTS | Too Many Requests|




[logo]: images/logo.png "Logo Title Text 2"
[warehouse]: images/warehouse.png "Warehouse image"
[shipment]: images/shipment.png "Describing Shipment"
[shipment_journey]: images/shipment_journey.png "Shipment Journey"
[shipment_reschedule_1]: images/shipment_reschedule_1.png "Shipment Reschedule Table 1"
[shipment_reschedule_2]: images/shipment_reschedule_2.png "Shipment Reschedule Table 2"



