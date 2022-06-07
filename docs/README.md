# Visitor API

This document contains the setup process for using the Konduko Visitor API.

Before using this documentation, you should have been sent an email with the required keys. If you have not received these keys, please email support@konduko.com

You should have received the following:

- Staging
  > **STAGING_EVENT_KEY**, **STAGING_API_KEY**
- Production
  > **PRODUCTION_EVENT_KEY**, **PRODUCTION_API_KEY**

We recommend that you do the following steps to integrate your system with the Konduko Visitor API:

1. Initial setup
2. Connect to Konduko's staging environment
3. Test a single Visitor upload
4. Test a bulk Visitor upload
5. Prepare for switching to the production environment
6. Bulk Upload Visitor records for your Event

# Initial Setup

Konduko's API sends and receives data in JSON format over HTTPS/REST.

For security reasons all connections must use HTTPS. Any calls made over insecure HTTP will be rejected with this response:

> HTTP/1.1 301 Moved Permanently

**NOTE**: Throughout this document, you will see references to "HTTP Headers", "HTTP/1.1", etc. These refer to the actual HTTP protocol, but connections must be encrypted (HTTPS) in all cases nonetheless.

## Production Url

https://prod-api.konduko.com/

## Staging Url

https://staging-api.konduko.com/

## Headers + API Key

API calls must include appropriate HTTP headers, including the API Key (provided by Konduko) which permits you to make those API calls.

The API Key is per-Event, which means that every Event will have its own unique API Key.

```
Content-Type: application/json
Accept: application/json
Authorization: Bearer YOUR_API_KEY_GOES_HERE
EventKey: EVENT_KEY_GOES_HERE
```

API calls made without correct authorisation will return this code:

> HTTP/1.1 401 Unauthorized

This may happen if:

- The API Key is missing from the Authorization: header
- The API Key is authorised for a different Event
- The API call was for an endpoint that is outside of the Visitor API scope

## JSON

Any data you wish to send with your API calls must formatted in JSON. For security reasons, the Konduko API only accepts data which is validated according to the strict [RFC 4627](https://www.ietf.org/rfc/rfc4627.txt) specification.

Any calls made with invalid JSON will return this code:

> HTTP/1.1 400 Bad Request

You can verify the validity of your JSON using online validation tools, such as http://jsonlint.com/

## Visitor API Endpoints

These are the endpoints involved in the Visitor API:

> POST /visitors

The JSON document for a request is what is found below:

```json
[
  {
    "bar_code": "string",
    "badge_mac": "string",
    "full_name": "string",
    "email": "string",
    "title": "string",
    "first_name": "string",
    "last_name": "string",
    "language": "string",
    "phone": "string",
    "job_title": "string",
    "company": "string",
    "department": "string",
    "address_one": "string",
    "address_two": "string",
    "city": "string",
    "state": "string",
    "postal_code": "string",
    "country": "string",
    "visitor_type": "string",
    "extra_data": {}
  }
]
```

Required fields are:

- **bar_code**
- **full_name**
- **first_name**
- **last_name**
- **email**

If any of the required fields are missing you will receive a

> HTTP/1.1 400 Bad Request

Fields with minimum requirements are:

- **bar_code** --> Should have a minimum of 5 characters
- **email** --> Disposable temporary emails like @yopmail.com are not allowed
- **country** --> The api uses ISO_3166 to validate countries and accepts name, alpha 2 and alpha 3 standards. 
- **badge_mac** --> Should be a valid mac address of 14 alphanumeric characters separated by colon. For instance: 3D:F2:C9:A6:B3:4F:FF.

If any of these requirements are not met, the api will return 

```json
[
  {
    "status_code": 400,
    "message": " FIELD NAME does not meet minimum requirements",
    "data": {
      "bar_code": "string",
      "badge_mac": "string"
    }
  }
]
```

If any of the required fields are missing you will receive a

> HTTP/1.1 400 Bad Request

The response to any valid request will be

```json
[
  {
    "status_code": 0,
    "message": "string",
    "data": {
      "bar_code": "string",
      "badge_mac": "string"
    }
  }
]
```

# Connect to the staging environment

Konduko provides a staging environment (see above) which can be used for setup and testing of your systems with the Konduko Visitor API.

Any information sent to this server is completely isolated from the live production environment and will not be used for any purposes other than for your own testing.

You will also have access to an Event on this same environment described in the Visitor API Credentials Register provided to you by Konduko.

> NOTE:

The **STAGING_API_KEY** should be placed in the Authorization: HTTP header for all API calls.

The **STAGING_EVENT_KEY** should be placed in the EventKey: HTTP header for all API calls.

# Test a single Visitor upload

## Example 1: POST a single Visitor record

Using the correct HTTP Headers, as described above, use your HTTP client to perform a

> POST https://staging-api.konduko.com/visitors/

You will need to send a JSON object as the content of your HTTP POST.

This JSON object must, at minimum, contain all required fields as specified above. Naturally, it is better to include as much Visitor information in the JSON object as possible

Here is an example curl command:

```console
curl -i -X POST
-H 'Content-Type: application/json'
-H 'Accept: application/json'
-H 'Authorization: Bearer {token}'
-H 'EventKey: XYZ-2015'
-d @visitor_data.json 'https://staging-api.konduko.com/visitors
```

Where visitor_data.json is:

```json
[
  {
    "bar_code": "12300001",
    "badge_mac": "3D:F2:C9:A6:B3:4F:FF",
    "first_name": "Test",
    "last_name": "User",
    "email": "test.user@testing.com"
  }
]
```

This will return

> HTTP/1.1 200

with the following body:

```json
[
  {
    "status_code": 200,
    "message": "OK",
    "data": {
      "bar_code": "12300001",
      "badge_mac": "3D:F2:C9:A6:B3:4F:FF"
    }
  }
]
```

If you send the same request again the following will be returned:

```json
[
  {
    "status_code": 200,
    "message": "Visitor Updated",
    "data": {
      "bar_code": "12300001",
      "badge_mac": "3D:F2:C9:A6:B3:4F:FF"
    }
  }
]
```

## Example 3: POST Visitor records in bulk

Posting Visitor records in bulk is exactly the same as Posting a single Visitor record, but where the JSON object has been extended to include multiple Visitor records:

Single Visitor Record

```json
[{ "bar_code": "12300001", "badge_mac": "3D:F2:C9:A6:B3:4F:FF" }]
```

Bulk Visitor Record

```json
[
  { "bar_code": "12300001", "badge_mac": "3D:F2:C9:A6:B3:4F:FF" },
  { "bar_code": "12300002", "badge_mac": "3D:F2:C9:A6:B3:4F:GG" },
  { "bar_code": "12300003", "badge_mac": "3D:F2:C9:A6:B3:4F:HH" }
]
```

## Example 3: Updating Visitor records

Updating Visitor records is exactly the same process as submitting them for the first time.

If you POST a Visitor record with a barcode that matches a Visitor that already exists in that Event, then any fields in your POST will overwrite any existing fields stored in the Konduko Cloud.

The sole exception to the above is that sending a Visitor record with a new NFC badge mac will add to the list of NFC Badges associated with that Visitor.

You can use this to, for example, correct a spelling mistake or add a new piece of information into a field that was previously blank.

Updating records can be done either individually, or in bulk.

# Error handling
Not only are errors returned via the HTTP methods, but the API returns a status for each of the individual records included in the bulk import.
This value is populated in the field "status_code" and is modelled off HTTP response codes.

We recommend looping through the responses to ensure that each visitor API record, has imported successfully.

A succcessful import will have the "status_code" field for example, populated with the value of 200, whereas an error during import, will have the error 500.
Other possible responses are the codes 400 and 409.

If the error 500 appears during import, please contact support@konduko.com.

# Prepare for switching to the production environment

Once your testing with the Visitor API on the staging environment of Konduko Cloud is complete, you can switch to using the live production environment.

Any changes you make to the production environment will impact your real users, and will affect the live Konduko Cloud. To minimise the chances of error, please check the following succeed before switching to the production environment:

Your system is designed to react appropriately to all of the following HTTP responses:

- HTTP/1.1 200 OK
- HTTP/1.1 301 Moved Permanently
- HTTP/1.1 400 Bad Request
- HTTP/1.1 401 Unauthorized
- HTTP/1.1 500 Server Error
- Your system can connect to the Visitor API with the API Keys provided in the Visitor API Credentials registered by Konduko
- POST Visitor record
- POST Visitor records in bulk
- POST Visitor record(s) to update them
- It is possible to post 1000 records in a bulk POST with acceptable performance.

# Bulk Upload Visitor records for your Event

You are now ready to bulk upload Visitors for your Event.

You will require two pieces of information, both of which will be provided by Konduko:

1. The **EVENT_KEY**, which goes in the header `EventKey`
2. The **API_KEY**, which goes in the HTTP Header `Authorization`
