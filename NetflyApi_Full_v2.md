# 📄 Netfly Peppol REST API – Documentation v2.1

Welcome to the Netfly Peppol REST API documentation. This API is designed to allow secure and efficient exchange of business documents (such as invoices) with the Peppol network. In addition to sending and receiving documents, it also allows clients to manage their own list of Peppol participants through a RESTful interface.

## ✨ Features

- 🔐 Token-based authentication via Auth0 (OAuth 2.0 Client Credentials)
- 📤 Send UBL-based XML documents into the Peppol network
- 📥 Retrieve incoming Peppol documents assigned to your organization
- 📃 Get a list of all sent and received documents
- 👥 Manage your own Peppol participants (create, update, delete, list)

## 🏗️ Architecture

The API is built on modern Jakarta EE standards and is protected with Auth0 JWT tokens. All endpoints accept and return data in JSON or XML format, depending on the operation. The system is designed to interoperate with ERP systems via REST calls, making integration flexible and secure.

## 🧾 Version

- API Version: 2.1
- Last Update: 12 April 2025

## 🧰 Requirements

- Auth0 Client ID and Secret (provided by Netfly)
- Proper Peppol-compliant XML document format (UBL 2.1 or equivalent)
- HTTPS connectivity to the Netfly API host

> ℹ️ Each request to the API must be authenticated using a valid JWT token obtained from the Auth0 token endpoint. See Authentication section for details.


---
# 🌍 API Environments: Test & Production

The Netfly Peppol REST API is available in two distinct environments:

- ✅ TEST Mode — for development, integration, and validation purposes.
- 🚀 PRODUCTION Mode — for live usage in production workflows.

Each environment has its own base URL, authentication domain, and credentials. Users will receive separate Client ID and Client Secret values for each mode.

## 🔐 Authentication Settings

When authenticating with Auth0, you must use the correct audience and token URL depending on the environment.

| Environment | Auth0 Domain                  | Audience URL                                             |
|-------------|-------------------------------|-----------------------------------------------------------|
| TEST        | https://netfly-test.eu.auth0.com | https://netfly-test.eu.auth0.com/api/v2/                |
| PRODUCTION  | https://netfly-go.eu.auth0.com  | https://netfly-go.eu.auth0.com/api/v2/                  |

> 🧠 Note: The audience must match exactly. Incorrect values will result in authentication failure.

## 🌐 REST API Base URLs

All endpoints are prefixed by a base URL that depends on the selected environment.

| Environment | API Base URL                          |
|-------------|----------------------------------------|
| TEST        | https://peppol2.netfly.be              |
| PRODUCTION  | https://service.netfly.eu.com          |

> ⚠️ Only use the Production environment once your integration has been approved by Netfly and is ready for live operations.

## 📋 Credentials Management

You will receive two sets of credentials:

- One Client ID and Client Secret for the TEST environment
- A separate Client ID and Client Secret for the PRODUCTION environment

Never mix credentials or use TEST credentials in production or vice versa.

## 🧪 Best Practices

- Test thoroughly in the TEST environment before switching to PRODUCTION.
- Validate all participant identifiers, payload formats, and headers.
- Ensure your server uses HTTPS for all requests.


---
# 🔐 Authentication via Auth0

Before accessing any Netfly Peppol API endpoints, you must authenticate via Auth0 using the OAuth 2.0 Client Credentials Flow.
This will provide you with a JWT (JSON Web Token) that must be included in the Authorization header of your API requests.

## ✅ Sample Authentication Request (curl)

Here is an example of how to retrieve your access token using curl:

```bash
curl --request POST \
  --url https://netfly-test.eu.auth0.com/oauth/token \
  --header 'Content-Type: application/json' \
  --data '{
    "client_id": "YOUR_CLIENT_ID",
    "client_secret": "YOUR_CLIENT_SECRET",
    "audience": "https://netfly-test.eu.auth0.com/api/v2/",
    "grant_type": "client_credentials"
  }'
```

Replace:

- YOUR_CLIENT_ID with the actual Client ID assigned to your application
- YOUR_CLIENT_SECRET with your corresponding secret

> 🔐 Note: Ensure you are using the correct audience and Auth0 domain that matches your environment (TEST or PRODUCTION). See [API Environments](#🌍-api-environments-test--production) for reference.

## 📥 Sample JSON Response

If the request is successful, Auth0 will return a JSON response like the following:

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...Abc123XYZ",
  "scope": "read:actions update:actions delete:actions create:actions",
  "expires_in": 86400,
  "token_type": "Bearer"
}
```

Only the access_token is required for calling the API.

## 🔄 Token Expiration

- The token is valid for 24 hours (86400 seconds).
- When it expires, you must request a new token using the same authentication request.

## 📎 Usage in API Calls

Once you have the token, include it in the Authorization header of your HTTP requests like this:

```http
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...Abc123XYZ
```

This ensures that your client is authenticated and authorized to interact with Netfly’s API endpoints.


---
# 📤 Send Business Document to Peppol Network

This endpoint allows an authenticated client to send a business document (e.g., Invoice or Credit Note) to the Peppol network.
The document must be conformant with the Peppol BIS Billing 3.0 standard and encoded in XML.
Authentication is required via a Bearer token issued by the Netfly Auth0 server.

### 🌐 Endpoint
`POST /netfly/sendDocument`

### Headers
- `Authorization: Bearer YOUR_ACCESS_TOKEN`
- `Content-Type: application/xml`

### Request Body
Binary XML file (UBL Invoice or Credit Note conformant with Peppol BIS Billing 3.0).

### 🧪 cURL Example
```bash
curl -X POST https://peppol2.netfly.be/netfly/sendDocument \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/xml" \
  --data-binary @path/to/invoice.xml
```

### ✅ Successful Response
```json
{
  "success": true,
  "message": "XML reception acknowledged",
  "code": "SA001"
}
```

### ❌ Unsuccessful Response
```json
{
  "success": false,
  "message": "undetermined UBL (unable to determine UBL)",
  "code": "SE003"
}
```

📘 This method is ideal for clients wishing to integrate their ERP system directly with the Peppol infrastructure using standardized UBL documents.


---
# 📄 Retrieve List of Business Documents

This endpoint allows a client to retrieve a list of business documents that have been sent to or received from the Peppol network.

The response contains detailed information for each document, such as:

- Internal ID
- Submission/Reception date
- Sender and Recipient Peppol Identifiers
- Original document ID
- Document type and VESID
- Country of origin
- Submission status
- Retry count and feedback
- Internal file name (used to retrieve the full XML document later)

⚠️ The internal document ID is required for fetching the full XML file via the receiveDocument endpoint.

### 🌐 Endpoint
`GET /netfly/documentsList`

### 🔧 Required Parameters

This endpoint supports the following query parameters:

| Parameter   | Description                              | Format              | Required |
|-------------|------------------------------------------|---------------------|----------|
| `startDate` | Start of search interval (UTC timestamp) | `yyyyMMddHHmmss`    | Yes      |
| `endDate`   | End of search interval (UTC timestamp)   | `yyyyMMddHHmmss`    | Yes      |
| `flow`      | Flow direction                           | `in` or `out`       | Yes      |

### 🧪 cURL Example

```bash
curl -X GET "https://peppol2.netfly.be/netfly/documentsList?startDate=20250201000000&endDate=20250430235959&flow=out" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### ✅ Successful Response

```json
[
  {
    "id": 6,
    "inputDate": "2025-03-16T09:16:40Z",
    "clientNumber": "0001",
    "sender": "9925:BE0475689186",
    "recipient": "9925:BE0426616985",
    "documentId": "Snippet_001",
    "documentType": "urn:fdc:peppol.eu:2017:poacc:billing:01:1.0",
    "documentVesid": "eu.peppol.bis3:invoice:latest-active",
    "country": "BE",
    "status": "OK",
    "feedback": "SUCCESSFULLY SUBMITTED TO AP",
    "retries": 1,
    "lastTry": "2025-03-16T09:18:56Z",
    "fileName": "fc7b5afc-173c-412e-ac66-0d27415a30f7.xml"
  }
]
```

### ❌ Sample Error or Retry Status

```json
[
  {
    "id": 7,
    "inputDate": "2025-04-12T20:03:25Z",
    "clientNumber": "0001",
    "sender": "9925:BE0475689186",
    "recipient": "9925:BE0426616985",
    "documentId": "Snippet1",
    "documentType": "urn:fdc:peppol.eu:2017:poacc:billing:01:1.0",
    "documentVesid": "eu.peppol.bis3:invoice:latest-active",
    "country": "BE",
    "status": "RETRY",
    "feedback": "SUBMISSION TO AP FAILED",
    "retries": 2,
    "lastTry": "2025-04-12T20:12:27Z",
    "fileName": "2d77c966-c85a-4a34-9d69-4ad063f0cffd.xml"
  }
]
```

### 📝 Notes

- The endpoint is read-only and supports only GET requests.
- If no documents match the criteria, an empty list is returned.
- This endpoint is useful to identify document submission status and track errors if any.


---
# 📥 Receive Business Document from Peppol Network

This endpoint allows a client to retrieve a previously received (or sent) Peppol BIS Billing 3.0 document in XML format from the Netfly Access Point.
Netfly’s Access Point (AP) can only receive documents on behalf of participants that are registered in an SMP (Service Metadata Publisher) pointing to Netfly. Optionally, clients can configure email notifications to be sent when a new document arrives.
This endpoint can also be used to retrieve a previously sent document by specifying the appropriate document ID and flow direction (`in` or `out`).

### 🌐 Endpoint
`GET /netfly/receiveDocument`

### 📥 Parameters

| Name     | Type   | Required | Description                                 |
|----------|--------|----------|---------------------------------------------|
| docId    | int    | Yes      | Internal file ID (as returned by documentsList) |
| flow     | string | Yes      | Either `in` or `out`                        |

### 🧪 cURL Example

```bash
curl -X GET "https://peppol2.netfly.be/netfly/receiveDocument?docId=1234&flow=in" \ 
     -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### ✅ Successful Response
Returns the full XML document in Peppol BIS Billing 3.0 format. Here’s a truncated example:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Invoice xmlns="urn:oasis:names:specification:ubl:schema:xsd:Invoice-2">
    <cbc:ID>Snippet1</cbc:ID>
    <cbc:IssueDate>2017-11-13</cbc:IssueDate>
    ...
</Invoice>
```

> ℹ️ Full XML structure is omitted in the documentation for brevity.

### ❌ Error Response

If the document is not found or does not belong to the authenticated client:

```json
{
  "success": false,
  "message": "no document found with the provided ID and client number",
  "code": "RE000"
}
```


---
# 📌 Add Participant

This endpoint allows the API user to register their own participants, who can then be declared as **senders** in Peppol-compliant business documents. For instance, the "supplier party" in a Peppol invoice must correspond to a participant previously declared by the user.
Netfly validates that each participant is registered in the Peppol network before allowing its creation via this endpoint.

### 🌐 Endpoint
`POST /netfly/participantManagement`

### 📥 Request Format

The request must be sent as JSON and include the following headers:

- `Authorization: Bearer YOUR_ACCESS_TOKEN`
- `Content-Type: application/json`

### 🧪 cURL Example

```bash
curl -X POST https://peppol2.netfly.be/netfly/participantManagement \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Company SA",
    "country": "BE",
    "participantId": "be0123456789",
    "participantScheme": "9925",
    "participantPrefix": "iso6523-actorid-upis",
    "webURI": "https://www.company.be",
    "contactType": "CEO",
    "contactName": "John Doe",
    "contactPhone": "+32 475 123456",
    "contactEmail": "john@company.be"
}'
```

### ✅ Successful Response

```json
{
  "success": true,
  "message": "Participant created successfully",
  "code": "PM000"
}
```

### ❌ Error Response Example

```json
{
  "success": false,
  "message": "Duplicate participant identifier",
  "code": "PMC06"
}
```

💡 Note: Ensure that all required fields are correctly formatted and that the participant already exists in the Peppol network (Netfly will perform this validation automatically).


---
# 📝 Update a Participant

This endpoint allows the API user to update an existing participant record. It is used when information such as the contact person, company name, or website needs to be changed. Netfly API verifies that the participant still exists and is valid in the Peppol network.

Just like the Add Participant endpoint, this API requires all participant information and validates it against the Peppol registry.

🔁 The participant must already exist in the database and must belong to the authenticated client.

### 🌐 Endpoint  
`PUT /netfly/participantManagement`

### 🧪 cURL Example

```bash
curl -X PUT https://peppol2.netfly.be/netfly/participantManagement \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Company SA",
    "country": "BE",
    "participantId": "be0123456789",
    "participantScheme": "9925",
    "participantPrefix": "iso6523-actorid-upis",
    "webURI": "https://www.company.be",
    "contactType": "OWNER",
    "contactName": "John Doe",
    "contactPhone": "+32 475 123456",
    "contactEmail": "john@company.be"
}'
```

### ✅ Successful Response

```json
{
  "success": true,
  "message": "Participant updated successfully",
  "code": "PM000"
}
```

### ❌ Unsuccessful Response

```json
{
  "success": false,
  "message": "Participant prefix is incorrect - must be iso6523-actorid-upis",
  "code": "PMU05"
}
```


---
# 🔍 Participants List

This endpoint allows an API user to retrieve a list of their registered participants. It is especially useful to verify or manage participant metadata such as contact information and registration identifiers.

### 🌐 Endpoint
`GET /netfly/participantsList`

### 📥 Optional Query Parameters

You can use any combination of the following parameters:

- `name` — Partial or full name of the participant (e.g., "Netfly")
- `contactName` — Partial or full contact person name (e.g., "Robert")
- `contactEmail` — Email address of the contact person
- `participantId` — Participant identifier (e.g., "be0475689186")

If no parameters are provided, all participants belonging to the authenticated client will be returned.

### 🧪 cURL Example

```bash
curl -X GET "https://peppol2.netfly.be/netfly/participantsList?name=Netfly&contactName=Robert&contactEmail=robert@netfly.be&participantId=be0475689186" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### ✅ Successful Response

```json
[
  {
    "id": 1,
    "name": "Netfly Srl",
    "country": "BE",
    "countryName": "BE",
    "participantId": "be0475689186",
    "participantScheme": "9925",
    "participantPrefix": "iso6523-actorid-upis",
    "webURI": "www.netfly.eu.com",
    "contactType": "Owner",
    "contactName": "Robert Kowinski",
    "contactPhone": "+32 486 243 141",
    "contactEmail": "robert@netfly.eu.com",
    "inputDate": [2025, 2, 14, 6, 50, 58],
    "clientNumber": "0001"
  }
]
```

The `inputDate` is returned as an array: `[year, month, day, hour, minute, second]`.

### ❌ Unsuccessful Response 

If something goes wrong (e.g., unauthorized request), a standard error object will be returned:

```json
{
  "success": false,
  "message": "internal server error",
  "code": "PL501"
}
```


---
# 🗑️ Delete a Participant

This endpoint allows the API user to delete a participant that belongs to them, by specifying the participant’s `id`.

⚠️ Only participants that belong to the authenticated client can be deleted.

### 🌐 Endpoint
`DELETE /netfly/participantManagement?id={participant_id}`

### 📥 Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| id   | int  | Yes      | ID of the participant to be deleted |

### 🧪 cURL Example

```bash
curl -X DELETE "https://peppol2.netfly.be/netfly/participantManagement?id=5" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### ✅ Successful Response

```json
{
  "success": true,
  "message": "Participant deleted successfully",
  "code": "PM013"
}
```

### ❌ Unsuccessful Response

```json
{
  "success": false,
  "message": "Participant not found or not owned by your client",
  "code": "PMD03"
}
```


---
# 📄✅ Validate a Peppol BIS Billing 3.0 Document

This special endpoint is a powerful tool for API users, enabling them to verify the compliance of their business documents—especially invoices—before submitting them to the Peppol network. It helps validate Peppol BIS Billing 3.0 (UBL format) against the applicable XML Schema and Schematron rulesets.

## Endpoint Description

The endpoint is used to validate Peppol BIS Billing 3.0 (latest version) documents in XML format.

- URL: `https://peppol2.netfly.be/docval/api/validate/eu.peppol.bis3:invoice:latest`
- Method: `POST`
- Content-Type: `application/xml`
- Header: `X-Token: SPECIAL_TOKEN`
- Body: Raw XML invoice document

### 🧪 cURL Example

```bash
curl -X POST \ 
  -H "Content-Type: application/xml"   -H "X-Token: SPECIAL_TOKEN"   -d @Path/To/invoice_from.xml   https://peppol2.netfly.be/docval/api/validate/eu.peppol.bis3:invoice:latest
```

### ✅ Response (Success)

```json
{
  "validationSource": {
    "sourceTypeID": "xml",
    "systemID": "uploaded content",
    "partialSource": false,
    "payloadBase64": "PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0i... (truncated for readability)"
  },
  "ves": {
    "vesid": "eu.peppol.bis3:invoice:2024.11",
    "name": "OpenPeppol UBL Invoice (2024.11) (aka BIS Billing 3.0.18)",
    "deprecated": false,
    "status": {
      "lastModification": "2025-04-13T23:07:42.075+02:00",
      "type": "valid",
      "validFrom": "2025-02-17T00:00:00Z"
    }
  },
  "success": true,
  "interrupted": false,
  "mostSevereErrorLevel": "SUCCESS",
  "results": [
    {
      "success": "TRUE",
      "artifactType": "xsd",
      "artifactPathType": "classpath",
      "artifactPath": "external/schemas/ubl21/maindoc/UBL-Invoice-2.1.xsd",
      "items": [],
      "durationMS": 2
    },
    {
      "success": "TRUE",
      "artifactType": "schematron-xslt",
      "artifactPathType": "classpath",
      "artifactPath": "external/schematron/openpeppol/2024.11/xslt/CEN-EN16931-UBL.xslt",
      "items": [],
      "durationMS": 17
    },
    {
      "success": "TRUE",
      "artifactType": "schematron-xslt",
      "artifactPathType": "classpath",
      "artifactPath": "external/schematron/openpeppol/2024.11/xslt/PEPPOL-EN16931-UBL.xslt",
      "items": [],
      "durationMS": 13
    }
  ],
  "durationMS": 33,
  "invocationDateTime": "2025-04-13T21:17:36.838910456Z",
  "invocationDurationMillis": 38
}
```

The validation report confirms compliance with Peppol standards if all rulesets pass successfully.

---
# 🚦 API Operation Result Codes

## Client Authentication/Authorization

| Code | Message | HTTP Status |
|------|---------|--------------|
| `AUTH001` | unauthorized: missing or invalid token | 401 |
| `AUTH002` | unauthorized: missing client_id claim | 401 |
| `AUTH501` | internal authentication error | 500 |
| `CE000` | client authentication failed | 403 |
| `CE001` | unknown client_id | 403 |
| `CE002` | multiple APIs assigned to your client_id | 403 |
| `CE003` | inactive client_id | 403 |
| `CE004` | organization not found for client_id | 403 |
| `CE005` | multiple organizations assigned to your client_id | 403 |

## Send Business Document to Peppol Network

| Code | Message | HTTP Status |
|------|---------|--------------|
| `SA001` | document approuved for sending | 200 |
| `SE000` | invalid HTTP Content-Type | 400 |
| `SE001` | not well-formed XML document | 400 |
| `SE002` | unsupported UBL | 400 |
| `SE003` | undetermined UBL (unable to determine UBL) | 400 |
| `SE004` | document validation failed | 400 |
| `SE005` | document Id is already used | 400 |
| `SE006` | participant not found in organization | 400 |
| `SE500` | Internal server error | 500 |
| `SE501` | failed to process request | 500 |
| `SE502` | failed to add to outgoing queue | 500 |
| `SE503` | failed to check document Id | 500 |
| `SE504` | unable to validate document | 500 |


## Receive Business Document from Peppol Network

| Code | Message | HTTP Status |
|------|---------|--------------|
| `RA000` | file returned to client | 200 |
| `RE000` | no document found with the provided ID | 404 |
| `RE001` | file not found on server | 404 |
| `RE002` | missing or empty 'docId' parameter | 400 |
| `RE003` | missing or empty 'flow' parameter | 400 |
| `RE500` | internal error | 500 |
| `RE501` | failed to process request | 500 |


## Retrieve List of Business Documents

| Code | Message | HTTP Status |
|------|---------|--------------|
| `LA000` | list returned to client | 200 |
| `LE000` | missing one or more required parameters: startDate, endDate, flow | 400 |
| `LE001` | date format must be yyyyMMddHHmmss | 400 |
| `LE002` | endDate must be greater than or equal to startDate | 400 |
| `LE003` | parameter flow must be one of [in, out] | 400 |
| `LE500` | internal error | 500 |
| `LE501` | failed to process request | 500 |


## Add Participant

| Code | Message | HTTP Status |
|------|---------|--------------|
| `PMC00` | participant created successfully | 200 |
| `PMC01` | missing required fields | 400 |
| `PMC02` | invalid ISO country code | 400 |
| `PMC03` | invalid email format | 400 |
| `PMC04` | participant scheme is inactive or unknown (example correct values: 0208, 9925, ...) | 400 |
| `PMC05` | participant prefix is incorrect (must be iso6523-actorid-upis) | 400 |
| `PMC06` | duplicate participant identifier | 400 |
| `PMC07` | failed to create participant | 500 |
| `PM500` | internal error | 500 |


## Update Participant

| Code | Message | HTTP Status |
|------|---------|--------------|
| `PMU00` | participant updated successfully | 200 |
| `PMU01` | missing required fields | 400 |
| `PMU02` | invalid ISO country code | 400 |
| `PMU03` | invalid email format | 400 |
| `PMU04` | participant scheme is inactive or unknown (example correct values: 0208, 9925, ...) | 400 |
| `PMU05` | participant prefix is incorrect (must be iso6523-actorid-upis) | 400 |
| `PMU06` | participant not found or not owned by specified client | 404 |
| `PMU07` | failed to update participant | 500 |
| `PM500` | internal error | 500 |


## Delete a Participant

| Code | Message | HTTP Status |
|------|---------|--------------|
| `PMD00` | participant deleted successfully | 200 |
| `PMD01` | missing participant ID for deletion | 400 |
| `PMD02` | invalid participant ID format | 400 |
| `PMD03` | participant not found or not owned by specified client | 404 |
| `PMD04` | failed to delete participant | 500 |
| `PM500` | internal error | 500 |


## Participants List

| Code | Message | HTTP Status |
|------|---------|--------------|
| `PLA00` | list returned to client | 200 |
| `PL501` | failed to retrieve list | 500 |
