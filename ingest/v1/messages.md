This endpoint is designed to accept multiple messages, split them into individual messages, and pass them through Listener. Outside of splitting the messages, no transformation occurs. Call this endpoint to send multiple message at once instead of individual messages.

If the request payload is a JSON array[^1], it is parsed and each entity of the array is treated as an individual message in Listener.  If the payload is *not* a JSON array, a `Delimiter` header must be sent to indicate which character should be used to split the payload into messages.

### Definition

```http
POST https://listener-ingest-services.teradata.com/v1/messages HTTP/1.1
Authorization: token SOURCE_SECRET_KEY
Content-Type: application/json
Delimiter: |
Sync: 1
```

Header        | Required | Sample Value       | Purpose
------        | -------- | ------------       | -------
Authorization | Required | `fhwb-afwfa-wefaf` | Source secret key that determines to which source the data is sent.
Delimiter     | Optional | `,`                | Instructs Ingest to split the request body into multiple messages by a delimiter character. Can be any character except these: `\/"''`. Ignored if `Content-Type` is JSON.
Content-Type  | Optional | `application/json` | Content type of the request body. If `application/json`, Ingest will parse an array into multiple messages.
Sync          | Optional | `1`                | Request a UUID from Ingest to ensure Kafka accepted the message. Value must be 1, or will be treated as async.

### Example Request

```bash
curl \
  -H "Content-Type: application/json" \
  -H "Authorization: token 0f6fd50e-9515-4730-92c0-2360b5af56d3" \
  -H "Sync: 1" \
  -H "Delimiter: ," \
  -X POST -d 'one,two,three' \
  -i \
  https://listener-ingest-services.teradata.com/v1/messages
```

### Example Success Response - All Messages Accepted, Sync Enabled in Request

```http
HTTP/1.1 201 Created
Content-Type: application/json
X-Total: 3
```
```json
[
  {"uuid":"15183920e250242ac11000000"},
  {"uuid":"15183920e250242ac11000001"},
  {"uuid":"15183920e250242ac11000002"}
]
```

### Example Error Response - Only 1 Message Accepted, Sync Enabled in Request

```http
HTTP/1.1 500 Created
Content-Type: application/json
X-Error: Server rejected it to avoid allocation error
X-Total: 1
```
```json
[
  {"uuid":"15183920e250242ac11000000"}
]
```

### Response Codes

Code | Meaning
---- | -------
201  | Message was successfully received.
401  | Authorization header was missing, or could not be found.
500  | Message body could not be read or Kafka could not be reached. When Sync enabled in request payload, X-Total and X-Error in response headers will indicate number of messages accepted and cause of internal server error. 

[^1] You cannot send a JSON object and expect each key to be treated as an individual message. You must send a JSON array.
