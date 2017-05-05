Sources are entities that collect streaming data. During data ingest, a secret key is used to sort incoming data to the appropriate source.

* [Source Object](#source-object)
* [Get a List of Sources](#get-a-list-of-sources)
* [Get a Source](#get-a-source)
* [Create a Source](#create-a-source)
* [Update a Source](#update-a-source)
* [Regenerate Source Secret](#regenerate-source-secret)
* [Delete a Source](#delete-a-source)

## Source Object

A source contains the following properties:

Property    | Type    | Example | Description
------------|---------|---------|------------
source_id   | string  | `758fbda4-accc-4f90-8f09-cc0a164c8c28` | ID of the Source in the form of a UUID.
owner       | array   | `["jd123456"]` | Array of user IDs who have owner permissions on the source.
created_at  | string  | `2015-07-04T10:20:00Z` | Timestamp of moment of creation.
created_by  | string  | `av012345` | ID of user who created the source.
updated_at  | string  | `2015-12-25T10:20:00Z` | Timestamp of moment of creation.
updated_by  | string  | `jd123456` | ID of user who last updated the source.
name        | string  | `My source` | Name of the source.
description | string  | `A superb source` | Metadata description of the source.
secret      | string  | `f8a9f620-e0e6-470b-a6b8-1f16b003c034` | Secret key used when streaming data to the Ingest API.
state       | string  | `1` | State of the source, enabled or disabled.
production  | boolean | `false` | Production state of the source, which is set to true when an assigned target is locked.
source_type | string  | `REST` | Type of the source, REST or MQTT. No value defaults to REST.
subscription_info | object | `{"broker": "ssl://localhost:8883", "topic": "My MQTT topic", "private_key" : \"XX...\", "certificate : \"YY...\"}` | MQTT subscription information containing broker URL, topic and optionally a private key and security certificate. Applicable only if source type is MQTT.

The properties *source_id*, *created_at*, *created_by*, *updated_at*, *updated_by* and *secret* are assigned by the Teradata Listener API at the moment of creation and are __Read-Only__.

The properties *owner* and *state* are assigned by the Teradata Listener API at the moment of creation.

The property *secret* is assigned by the Teradata Listener API at the moment of creation and is __Read-Only__ and only visible to *owner*.

The property *production* is inherited from the associated Target resource and is __Read-Only__.

The property *state* can be changed only by deleting an item.

## Get a List of Sources

Load a list of all sources in Listener.

#### Definition

```http
GET https://listener-app-services.teradata.com/v1/sources?starred HTTP/1.1
```

Parameter | Required | Type    | Sample   | Purpose
--------- | -------- | ----    | -------- | -------
`starred` | Optional | boolean | `true`   | Filter the list of sources by the ones you sorted.

#### Example Request

```bash
curl \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -i \
  https://listener-app-services.teradata.com/v1/sources
```

#### Example Response

```http
HTTP/1.1 200 OK
Content-Type: application/json
```
```json
[
  {
    "source_id": "758fbda4-accc-4f90-8f09-cc0a164c8c28",
    "owner": ["jd123456"],
    "created_at": "2015-07-04T10:20:00Z",
    "created_by": "av012345",
    "updated_at": "2015-12-20T10:20:00Z",
    "updated_by": "jd123456",
    "secret": "f8a9f620-e0e6-470b-a6b8-1f16b003c034",
    "name": "My source",
    "description": "A superb source",
    "state": 1,
    "production": false,
    "source_type": "REST"
  },
  {
    "source_id": "758fbda4-accc-4f90-8f09-cc0a164c8c29",
    "owner": ["jd123456"],
    "created_at": "2015-07-04T10:21:00Z",
    "created_by": "av012345",
    "updated_at": "2015-12-20T10:21:00Z",
    "updated_by": "jd123456",
    "secret": "f8a9f620-e0e6-470b-a6b8-1f16b003c035",
    "name": "My other source",
    "description": "Another superb source",
    "state": 1,
    "production": false,
    "source_type": "MQTT",
    "subscription_info": {
      "broker": "http://localhost:1234",
      "topic": "My MQTT topic",
      "state": "connected"
    }
  },
  {
    "source_id": "758fbda4-accc-4f90-8f09-cc0a164c8c30",
    "owner": ["jd123456"],
    "created_at": "2015-07-04T10:22:00Z",
    "created_by": "av012345",
    "updated_at": "2015-12-20T10:22:00Z",
    "updated_by": "jd123456",
    "secret": "f8a9f620-e0e6-470b-a6b8-1f16b003c036",
    "name": "My other source 2",
    "description": "Another superb source 2",
    "state": 1,
    "production": false,
    "source_type": "MQTT",
    "subscription_info": {
      "broker": "http://localhost:1235",
      "topic": "My MQTT topic 2",
      "state": "disconnected",
      "reason": "Error connecting to subscriber or no subscriptions found."
    }
  },
  ...
]
```

#### Response Codes

Code | Meaning
---- | -------
201  | Sources were returned successfully.
401  | 401_STATUS

## Get a Source

Get a single source by a given source id.

#### Definition

```http
GET https://listener-app-services.teradata.com/v1/sources/{source_id} HTTP/1.1
```

#### Example Request

```bash
curl \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -i \
  https://listener-app-services.teradata.com/v1/sources/758fbda4-accc-4f90-8f09-cc0a164c8c28
```

#### Example REST Source Response

```http
HTTP/1.1 200 OK
Content-Type: application/json
```
```json
{
  "source_id": "758fbda4-accc-4f90-8f09-cc0a164c8c28",
  "owner": ["jd123456"],
  "created_at": "2015-07-04T10:20:00Z",
  "created_by": "av012345",
  "updated_at": "2015-12-20T10:20:00Z",
  "updated_by": "jd123456",
  "secret": "f8a9f620-e0e6-470b-a6b8-1f16b003c034",
  "name": "My source",
  "description": "A superb source",
  "state": 1,
  "production": false,
  "source_type": "REST"
}
```

#### Example MQTT Source Response

```http
HTTP/1.1 200 OK
Content-Type: application/json
```
```json
{
  "source_id": "758fbda4-accc-4f90-8f09-cc0a164c8c29",
  "owner": ["jd123456"],
  "created_at": "2015-07-04T10:21:00Z",
  "created_by": "av012345",
  "updated_at": "2015-12-20T10:21:00Z",
  "updated_by": "jd123456",
  "secret": "f8a9f620-e0e6-470b-a6b8-1f16b003c035",
  "name": "My other source",
  "description": "Another superb source",
  "state": 1,
  "production": false,
  "source_type": "MQTT",
  "subscription_info": {
    "broker": "http://localhost:1234",
    "topic": "My MQTT topic",
    "state": "connected"
  }
}
```

#### Response Codes

Code | Meaning
---- | -------
201  | Source was returned successfully.
401  | 401_STATUS
404  | 404_STATUS

## Create a Source

To create a new Source, provide a JSON object of the properties for the new source. Read-only properties are ignored.

**Note:** MQTT sources have been validated against ActiveMQ 5.14.0 and Mosquitto 1.3.4.

#### Definition

```http
POST https://listener-app-services.teradata.com/v1/sources HTTP/1.1
```

#### Example REST Source Request

```bash
curl \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -X POST \
  -d '{
    "name": "My source",
    "description": "A superb source",
    "source_type": "REST"
  }' \
  -i \
  https://listener-app-services.teradata.com/v1/sources
```

#### Example MQTT Source Request

```bash
curl \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -X POST \
  -d '{
    "name": "My source",
    "description": "A superb source",
    "source_type": "MQTT",
    "subscription_info": {
      "broker": "http://localhost:1234",
      "topic": "My MQTT topic",
      "certificate": "-----BEGIN CERTIFICATE-----\nMIID...",
      "private_key": "-----BEGIN RSA PRIVATE KEY-----\nMIIE..."
    }
  }' \
  -i \
  https://listener-app-services.teradata.com/v1/sources
```

#### Example REST Source Response

```http
HTTP/1.1 200 OK
Content-Type: application/json
```
```json
{
  "source_id": "758fbda4-accc-4f90-8f09-cc0a164c8c28",
  "owner": ["jd123456"],
  "created_at": "2015-07-04T10:20:00Z",
  "created_by": "av012345",
  "updated_at": "2015-12-20T10:20:00Z",
  "updated_by": "jd123456",
  "secret": "f8a9f620-e0e6-470b-a6b8-1f16b003c034",
  "name": "My source",
  "description": "A superb source",
  "state": 1,
  "production": false,
  "source_type": "REST"
}
```

#### Example MQTT Source Response

```http
HTTP/1.1 200 OK
Content-Type: application/json
```
```json
{
  "source_id": "758fbda4-accc-4f90-8f09-cc0a164c8c28",
  "owner": ["jd123456"],
  "created_at": "2015-07-04T10:20:00Z",
  "created_by": "av012345",
  "updated_at": "2015-12-20T10:20:00Z",
  "updated_by": "jd123456",
  "secret": "f8a9f620-e0e6-470b-a6b8-1f16b003c034",
  "name": "My source",
  "description": "A superb source",
  "state": 1,
  "production": false,
  "source_type": "MQTT",
  "subscription_info": {
    "broker": "http://localhost:1234",
    "topic": "My MQTT topic",
    "certificate": "-----BEGIN CERTIFICATE-----\nMIID...",
    "private_key": "-----BEGIN RSA PRIVATE KEY-----\nMIIE..."
  }
}
```

#### Response Codes

Code | Meaning
---- | -------
201  | Source was created successfully.
400  | A required property is missing from the request.
401  | 401_STATUS

## Update a Source

To update a source, send a JSON object with updated values for one or more of the __writable__ source properties.  Read-only fields are ignored. All property values from the previous version of this source are carried over by default, if not included in the hash.

#### Definition

```http
PATCH https://listener-app-services.teradata.com/v1/sources/{source_id} HTTP/1.1
```

#### Example REST Source Request

```bash
curl \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -X PATCH \
  -d '{
    "owner": ["jd123456"],
    "created_at": "2015-07-04T10:20:00Z",
    "created_by": "av012345",
    "updated_at": "2015-12-20T10:20:00Z",
    "updated_by": "jd123456",
    "secret": "f8a9f620-e0e6-470b-a6b8-1f16b003c034",
    "name": "My source",
    "description": "A superb source",
    "state": 1,
    "production": false,
    "source_type": "REST"
  }' \
  -i \
  https://listener-app-services.teradata.com/v1/sources/758fbda4-accc-4f90-8f09-cc0a164c8c28
```

#### Example MQTT Source Request

```bash
curl \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -X PATCH \
  -d '{
    "owner": ["jd123456"],
    "created_at": "2015-07-04T10:20:00Z",
    "created_by": "av012345",
    "updated_at": "2015-12-20T10:20:00Z",
    "updated_by": "jd123456",
    "secret": "f8a9f620-e0e6-470b-a6b8-1f16b003c034",
    "name": "My source",
    "description": "A superb source",
    "state": 1,
    "production": false,
    "source_type": "MQTT",
    "subscription_info": {
      "broker": "http://localhost:1234",
      "topic": "My MQTT topic"
    }
  }' \
  -i \
  https://listener-app-services.teradata.com/v1/sources/758fbda4-accc-4f90-8f09-cc0a164c8c28
```

#### Example REST Source Response

```http
HTTP/1.1 200 OK
Content-Type: application/json
```
```json
{
  "source_id": "758fbda4-accc-4f90-8f09-cc0a164c8c28",
  "owner": ["jd123456"],
  "created_at": "2015-07-04T10:20:00Z",
  "created_by": "av012345",
  "updated_at": "2015-12-20T10:20:00Z",
  "updated_by": "jd123456",
  "secret": "f8a9f620-e0e6-470b-a6b8-1f16b003c034",
  "name": "My source",
  "description": "A superb source",
  "state": 1,
  "production": false,
  "source_type": "REST"
}
```

#### Example MQTT Source Response

```http
HTTP/1.1 200 OK
Content-Type: application/json
```
```json
{
  "source_id": "758fbda4-accc-4f90-8f09-cc0a164c8c28",
  "owner": ["jd123456"],
  "created_at": "2015-07-04T10:20:00Z",
  "created_by": "av012345",
  "updated_at": "2015-12-20T10:20:00Z",
  "updated_by": "jd123456",
  "secret": "f8a9f620-e0e6-470b-a6b8-1f16b003c034",
  "name": "My source",
  "description": "A superb source",
  "state": 1,
  "production": false,
  "source_type": "MQTT",
  "subscription_info": {
    "broker": "http://localhost:1234",
    "topic": "My MQTT topic"
  }
}
```

#### Response Codes

Code | Meaning
---- | -------
201  | Source was updated successfully.
400  | A required property is missing from the request.
401  | 401_STATUS
404  | 404_STATUS

## Regenerate Source Secret Key

The source secret key is a private key that ensures only the desired data is ingested for a source. If the secret key is compromised, you can regenerate the secret key to replace the existing key.
Note that the source secret key is only available for REST sources (source_type of "REST" or null).

#### Definition

```http
PUT https://listener-app-services.teradata.com/v1/sources/{source_id} HTTP/1.1
```

#### Example Request

```bash
curl \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -X PUT \
  -i \
  https://listener-app-services.teradata.com/v1/sources/758fbda4-accc-4f90-8f09-cc0a164c8c28
```

#### Example Response

```http
HTTP/1.1 200 OK
Content-Type: application/json
```
```json
{
  "source_id": "758fbda4-accc-4f90-8f09-cc0a164c8c28",
  "owner": ["jd123456"],
  "created_at": "2015-07-04T10:20:00Z",
  "created_by": "av012345",
  "updated_at": "2015-12-20T10:20:00Z",
  "updated_by": "jd123456",
  "secret": "d8e38620-3729-57b1-a836-791320cef372",
  "name": "My source",
  "description": "A superb source",
  "state": 1,
  "production": false,
  "source_type": "REST"
}
```

#### Response Codes

Code | Meaning
---- | -------
201  | Source was created.
400  | A required property is missing from the request or the source type is not supported.
401  | 401_STATUS
403  | Cannot regenerate because source or target is locked.
404  | 404_STATUS

## Delete a Source

Only an owner can delete a source. When a source is deleted, its state is set to `0` and will not appear for users.

#### Definition

```http
DELETE https://listener-app-services.teradata.com/v1/sources/{source_id} HTTP/1.1
```

#### Example Request

```bash
curl \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -X DELETE \
  -i \
  https://listener-app-services.teradata.com/v1/sources/758fbda4-accc-4f90-8f09-cc0a164c8c28
```

#### Example Response

```http
HTTP/1.1 204 No Content
```

#### Response Codes

Code | Meaning
---- | -------
204  | Source was deleted successfully.
400  | Cannot delete a source associated with a target.
401  | 401_STATUS
404  | 404_STATUS
