Targets are the entities that process a stream from a source and manage how that data is persisted to a system. They have an attached entity called a "writer," which does the actual work for pulling data from Listener and storing it in the system.

* [Target Object](#target-object)
* [Target Types](#target-types)
* [Target Mapping](#target-types)
* [Get a List of Targets](#get-a-list-of-targets)
* [Get a Target](#get-a-target)
* [Create a Target](#create-a-target)
* [Update a Target](#update-a-target)
* [Delete a Target](#delete-a-target)
* [Get Target Columns](#get-target-columns)
* [Start a Target](#start-a-target)
* [Pause a Target](#pause-a-target)
* [Get Target Status](#get-target-status)
* [Validate a Target](#validate-a-target)
* [Retrieve Count of Dead Letter Records](#retrieve-count-of-dead-letter-records)
* [Retrieve Dead Letter Records](#retrieve-dead-letter-records)
* [Clear Dead Letter Queue](#clear-dead-letter-queue)

## Target Object

A target contains the following properties:

Property        | Type    | Example | Description
--------------- | ------- | ------- | -----------
target_id       | string  | `758fbda4-accc-4f90-8f09-cc0a164c8c28` | ID of the Target.
source_id       | string  | `84757abc-ba43-5736-0ba3-1bdac4748290` | ID of the Source associated with this target.
system_id       | string  | `9dda5570-70e0-45be-8449-83f33320cd08` | ID of the System associated with this target.
owner           | array   | `["jd123456"]` | ID of the all user who have owner permissions.
created_at      | string  | `2015-07-04T10:20:00Z` | Timestamp of moment of creation of the target.
created_by      | string  | `av012345` | ID of user that created the target.
updated_at      | string  | `2015-12-25T10:20:00Z` | Timestamp of moment of creation of the target.
updated_by      | string  | `jd123456` | ID of user who last updated the target.
name            | string  | `My Data Target` | Name of the target.
description     | string  | `A superb data target` | Description of the target.
target_type     | string  | `teradata` | Type of target. Must match the associated system type.
data_path       | object  | `{"schema": "listener": "table": "demo"}` | Specific data path on target system. Details vary by target type.
system_info     | object  | `{"password": "ip112233", "username": "8080"}` | Specific target system connection information. Details vary by target type.
state           | string  | `1` | State of the target, enabled or disabled.
production      | boolean | `false` | Production state of the target. When enabled, prevents modifications.
bundle          | boolean | `true` | Flag to indicate the target is batched (`true`) or realtime (`false`).
bundle_type     | records | `records` | When batch mode, declare mode using `records`. Other modes may be supported later.
bundle_interval | integer | `200` | When batch mode, declare the number of records during a batch.
data_map        | object  | `{ "mapping":[{"field": "uuid", "column": "uuid", "type": "varchar"}] }` | Describes how to map target fields to system columns.
properties      | object  | `{"webhcat_table": "listener"}` | Additional properties for some target types.
use\_dead\_letter\_queue | boolean | `true` | Whether to move "bad" records to a separate queue (`true`) or not (`false`).
dead\_letter\_queue | string | `b37979f6-b712-4d81-990e-9556a3c0e94b-dlq` | Name of the queue used to store failed records.

The properties *target_id*, *created_at*, *created_by*, *updated_at*, and *updated_by* are assigned by the Teradata Listener API at the moment of creation and are __Read-Only__.

The properties *owner*, *sample_size*, and *state* are assigned by the Teradata Listener API at the moment of creation.

The property *secret* is assigned by the Teradata Listener API at the moment of creation and is __Read-Only__ and only visible to *owner*.

The property *production* is governed by TBD and is __Read-Only__.

The property *state* can be changed only by deleting an item.

The property *dead\_letter\_queue* is __Read-Only__ and automatically populated if a target is configured with a *use\_dead\_letter\_queue* value of *true*.

## Target Mapping

Targets that support a database column structure can map fields from Listener to specific columns. All targets, except HDFS types, support mapping fields.

The mapping of data to columns can take one of two forms

1. Passthrough mapping
2. Shredding mapping

### Passthrough Mapping

Listener wraps messages with additional metadata that you can map and store in your target. The following four fields can be mapped:

Field     | Contents
-----     | --------
data      | Field with the raw data sent to Ingest.
source_id | ID for the source to which the packet was sent.
time      | Time the packet was ingested.
uuid      | UUID that gets assigned to the packet upon arrival.

Suppose there is a table with the following columns and associated names and types. Because the purpose of mapping is to allow you to link a particular field from above into a particular column, these could be named anything.

Column    | Type
------    | ----
records   | json
stream    | varchar
timestamp | timestamp
id        | varchar

The `data_map.mapping` array takes an object with three properties, `column`, `field`, and `type`. All three must be specified to validate. If you do not want to map a particular field, leave the object out of the array. Using the fields and the columns above, mapping would be defined in a target like the following example.

```json
{
  "data_map": {
    "mapping": [
      {
        "column": "records",
        "field": "data",
        "type": "json"
      }, {
        "column": "stream",
        "field": "source_id",
        "type": "varchar"
      }, {
        "column": "timestamp",
        "field": "time",
        "type": "timestamp"
      }, {
        "column": "id",
        "field": "uuid",
        "type": "varchar"
      }
    ]
  }
}
```

### Shredding Mapping

Listener ingests `JSON` data, the nature of `JSON` data allows for easy parsing and the potential to map from `JSON` field names to database columns. This functionality is exposed in Listener through the `data_map.mapping_type`.

```json
{
  "data_map": {
    "mapping_type" : "auto_shred"
  }
}
```

If a target is configured with `auto_shred` and a user provides the following `JSON`

```json
{
  "country" : "France",
  "population" : 66991000,
  "famous_for" : ["Wine", "Cheese"],
  "religions" : {
    "Christian" : 63,
	"None" : 25,
	"Muslim" : 9,
	"Other" : 3
  },
  "monarch" : null
}
```

The writer process will parse the provided JSON and make the following fields and values available for binding to the target database table.

Field        | Type        | Contents
-----        | ----        | ----
country      | Parsed      | "France"
population   | Parsed      | 66991000
famous\_for  | Parsed      | "[\"Wine\",\"Cheese\"]"
religions    | Parsed      | "{\"Christian\": 63, \"None\": 25, \"Muslim\": 9, \"Other\": 3}"
monarch      | Parsed      | null
data         | Listener    | Field with the raw data sent to Ingest.
source_id    | Listener    | ID for the source to which the packet was sent.
time         | Listener    | Time the packet was ingested.
uuid         | Listener    | UUID that gets assigned to the packet upon arrival.

The binding of values happens by examining the name of each column in the database table and finding the corresponding ```JSON``` field by comparing names. For example

Table Column  | Value                       | Explanation
---           | ---                         | ---
COUNTRY       | France                      | Matches `country` field through case insensitive comparison
UUID          | UUID associated with packet | Matches `uuid` field through case insensitive comparison
timezone      | null                        | No value in fields so defaults to null

## Target Types

Listener supports five types of targets. These targets have slightly different configuration options.

* [Teradata](#teradata)
* [Aster](#aster)
* [HDFS](#hdfs)
* [HBase](#hbase)
* [Websocket](#websocket)

#### Teradata

A Teradata target requires a `schema` and `table` in the `data_path`, as well as a `username` and `password` in the `system_info`.

```json
{
  "target_type": "teradata",
  "data_path": {
    "schema": "schema",
    "table": "table"
  },
  "data_map": {
    "mapping": [
      {
        "column": "db_column_name",
        "field": "data",
        "type": "json"
      }
    ]
  },
  "system_info": {
    "username": "ip112233",
    "password": "password"
  }
}
```

#### Aster

An Aster target requires a `schema` and `table` in the `data_path`, as well as a `username` and `password` in the `system_info`.

```json
{
  "target_type": "aster",
  "data_path": {
    "schema": "schema",
    "table": "table"
  },
  "data_map": {
    "mapping": [
      {
        "column": "db_column_name",
        "field": "data",
        "type": "json"
      }
    ]
  },
  "system_info": {
    "username": "ip112233",
    "password": "password"
  }
}
```

#### HDFS

An HDFS target requires a `path` and `extension` in the `data_path`, as well as a `username` and `password` in the `system_info`. Currently, only sequence files are supported, therefore, the extension `extension` must be `seq`.

```json
{
  "target_type": "hdfs",
  "data_path": {
    "path": "/data/listener/",
    "extension": "seq"
  },
  "properties": {
    "webhcat_table": "listener"
  },
  "system_info": {
    "username": "ip112233",
    "password": "password"
  }
}
```

#### HBase

A HBase target requires a `schema` (also referred to as a column family) and `table` in the `data_path`, as well as a `username` and `password` in the `system_info`.

```json
{
  "target_type": "hbase",
  "data_path": {
    "schema": "schema",
    "table": "table"
  },
  "data_map": {
    "mapping": [
      {
        "column": "db_column_name",
        "field": "data",
        "type": "json"
      }
    ]
  },
  "system_info": {
    "username": "ip112233",
    "password": "password"
  }
}
```

#### Websocket

The websocket type target will create a websocket server that allows the user to connect third party apps or custom logic
to a source in listener.

A websocket target simply requires a `source_id` and `name`.

```json
{
  "target_type": "websocket",
  "source_id": "84757abc-ba43-5736-0ba3-1bdac4748290",
	"name": "my websocket"
}
```

The elements returned when a websocket target is created will provide client connection details back to the user:

```json
{
	"data_path": {
		"secret": "ffafef69-43cf-44b8-a3c4-80a93808ac4f",
		"url": "https://listener-streamer-services-myenv.example.com/v1/streamer/e5925443-ec1c-4fd4-ada8-8d8c6c603cee",
		"websocket": "wss://listener-streamer-services-myenv.example.com/v1/streamer/e5925443-ec1c-4fd4-ada8-8d8c6c603cee"
	},
	"name": "my websocket",
	"source_id": "96bcaa82-6091-4e96-aee5-cc3fe2015bae",
	"target_id": "e5925443-ec1c-4fd4-ada8-8d8c6c603cee",
	"target_type": "websocket"
}
```

In order to connect to a websocket, you must send a websocket handshake request to the returned URL with the correct authorization header-

```
GET https://listener-streamer-services-myenv.example.com/v1/streamer/e5925443-ec1c-4fd4-ada8-8d8c6c603cee
Authorization: token ffafef69-43cf-44b8-a3c4-80a93808ac4f
```

A successful response means that you can then connect to the secure websocket at

```
wss://listener-streamer-services-myenv.example.com/v1/streamer/e5925443-ec1c-4fd4-ada8-8d8c6c603cee
```


## Get a List of Targets

Load a list of all targets in Listener.

#### Definition

```http
GET https://listener-app-services.teradata.com/v1/targets?source_id HTTP/1.1
```

Parameter   | Required | Type   | Sample   | Purpose
---------   | -------- | ----   | -------- | -------
`source_id` | Optional | string | `84757abc-ba43-5736-0ba3-1bdac4748290`| Filter the list of targets by those associated with a source.

#### Example Request

```bash
curl \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -i \
  https://listener-app-services.teradata.com/v1/targets
```

#### Example Response

```http
HTTP/1.1 200 OK
Content-Type: application/json
```
```json
[
  {
    "target_id": "758fbda4-accc-4f90-8f09-cc0a164c8c28",
    "source_id": "84757abc-ba43-5736-0ba3-1bdac4748290",
    "system_id": "9dda5570-70e0-45be-8449-83f33320cd08",
    "owner": ["jd123456"],
    "created_at": "2015-07-04T10:20:00Z",
    "created_by": "av012345",
    "updated_at": "2015-12-20T10:20:00Z",
    "updated_by": "jd123456",
    "name": "My Data Target",
    "description": "A superb data target",
    "target_type": "teradata",
    "data_path": {
      "schema": "schema",
      "table": "table"
    },
    "system_info": {
      "username": "ip112233",
      "password": "password"
    },
    "state": 1,
    "production": false,
    "bundle": true,
    "bundle_type": "records",
    "bundle_interval": 500
  }
]
```

#### Response Codes

Code | Meaning
---- | -------
201  | Targets were returned successfully.
401  | Authorization header was missing or could not be found.








## Get a Target

Get a single target by a given target id.

#### Definition

```http
GET https://listener-app-services.teradata.com/v1/targets/{target_id} HTTP/1.1
```

#### Example Request

```bash
curl \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -i \
  https://listener-app-services.teradata.com/v1/targets/758fbda4-accc-4f90-8f09-cc0a164c8c28
```

#### Example Response

```http
HTTP/1.1 200 OK
Content-Type: application/json
```
```json
{
  "target_id": "758fbda4-accc-4f90-8f09-cc0a164c8c28",
  "source_id": "84757abc-ba43-5736-0ba3-1bdac4748290",
  "system_id": "9dda5570-70e0-45be-8449-83f33320cd08",
  "owner": ["jd123456"],
  "created_at": "2015-07-04T10:20:00Z",
  "created_by": "av012345",
  "updated_at": "2015-12-20T10:20:00Z",
  "updated_by": "jd123456",
  "name": "My Data Target",
  "description": "A superb data target",
  "target_type": "teradata",
  "data_path": {
    "schema": "schema",
    "table": "table"
  },
  "system_info": {
    "username": "ip112233",
    "password": "password"
  },
  "state": 1,
  "production": false,
  "bundle": true,
  "bundle_type": "records",
  "bundle_interval": 500
}
```

#### Response Codes

Code | Meaning
---- | -------
201  | Target was returned successfully.
401  | Authorization header was missing or could not be found.
404  | The resource was not found.








## Create a Target

To create a new target, provide a JSON object of the properties for the new target. If read-only properties are supplied, they will be ignored. A target must be associated with a source and system, and include proper connection details. The user must also be an owner of the associated source to create a target.

#### Definition

```http
POST https://listener-app-services.teradata.com/v1/targets HTTP/1.1
```

#### Example Request

```bash
curl \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -X POST \
  -d '{
    "source_id": "84757abc-ba43-5736-0ba3-1bdac4748290",
    "system_id": "9dda5570-70e0-45be-8449-83f33320cd08",
    "name": "My Data Target",
    "description": "A superb data target",
    "target_type": "teradata",
    "data_path": {
      "schema": "schema",
      "table": "table"
    },
    "system_info": {
      "username": "ip112233",
      "password": "password"
    },
    "production": false,
    "bundle": true,
    "bundle_type": "records",
    "bundle_interval": 500
  }' \
  -i \
  https://listener-app-services.teradata.com/v1/targets
```

#### Example Response

```http
HTTP/1.1 200 OK
Content-Type: application/json
```
```json
{
  "target_id": "758fbda4-accc-4f90-8f09-cc0a164c8c28",
  "source_id": "84757abc-ba43-5736-0ba3-1bdac4748290",
  "system_id": "9dda5570-70e0-45be-8449-83f33320cd08",
  "owner": ["jd123456"],
  "created_at": "2015-07-04T10:20:00Z",
  "created_by": "av012345",
  "updated_at": "2015-12-20T10:20:00Z",
  "updated_by": "jd123456",
  "name": "My Data Target",
  "description": "A superb data target",
  "target_type": "teradata",
  "data_path": {
    "schema": "schema",
    "table": "table"
  },
  "system_info": {
    "username": "ip112233",
    "password": "password"
  },
  "state": 1,
  "production": false,
  "bundle": true,
  "bundle_type": "records",
  "bundle_interval": 500
}
```

#### Response Codes

Code | Meaning
---- | -------
201  | Target was created successfully.
400  | A required property is missing from the request.
401  | Authorization header was missing or user is not an owner of source.








## Update a Target

To update a target, send a JSON object with updated values for one or more of the __writable__ target properties.  If Read-only fields are supplied, they will be ignored. All property values from the previous version of this target are carried over by default if not included in the hash.

#### Definition

```http
PATCH https://listener-app-services.teradata.com/v1/targets/{target_id} HTTP/1.1
```

#### Example Request

```bash
curl \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -X PATCH \
  -d '{
    "production": true
  }' \
  -i \
  https://listener-app-services.teradata.com/v1/targets/758fbda4-accc-4f90-8f09-cc0a164c8c28
```

#### Example Response

```http
HTTP/1.1 200 OK
Content-Type: application/json
```
```json
{
  "target_id": "758fbda4-accc-4f90-8f09-cc0a164c8c28",
  "source_id": "84757abc-ba43-5736-0ba3-1bdac4748290",
  "system_id": "9dda5570-70e0-45be-8449-83f33320cd08",
  "owner": ["jd123456"],
  "created_at": "2015-07-04T10:20:00Z",
  "created_by": "av012345",
  "updated_at": "2015-12-20T10:20:00Z",
  "updated_by": "jd123456",
  "name": "My Data Target",
  "description": "A superb data target",
  "target_type": "teradata",
  "data_path": {
    "schema": "schema",
    "table": "table"
  },
  "system_info": {
    "username": "ip112233",
    "password": "password"
  },
  "state": 1,
  "production": true,
  "batch": true,
  "bundle_type": "records",
  "bundle_interval": 500
}
```

#### Response Codes

Code | Meaning
---- | -------
201  | Target was updated successfully.
400  | A required property is missing from the request.
401  | Authorization header was missing or could not be found.
404  | The resource was not found.







## Delete a Target

Only an owner or admin can delete a target, and doing so will set the state to `0`. Once a target is deleted, it will not appear for users the writer will be destroyed.

#### Definition

```http
DELETE https://listener-app-services.teradata.com/v1/targets/{target_id} HTTP/1.1
```

#### Example Request

```bash
curl \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -X DELETE \
  -i \
  https://listener-app-services.teradata.com/v1/targets/758fbda4-accc-4f90-8f09-cc0a164c8c28
```

#### Example Response

```http
HTTP/1.1 204 No Content
```

#### Response Codes

Code | Meaning
---- | -------
204  | Target was deleted successfully.
401  | Authorization header was missing or could not be found.
403  | Cannot delete a target marked with a production flag.
404  | The resource was not found.







## Start a Target

Only the owner or admin can start a target, and doing so will deploy a new writer that will begin to pull data from the stream and write to the target. If applicable, it confirms target columns are valid before starting, and sends an error if they are not valid.

#### Definition

```http
PUT https://listener-app-services.teradata.com/v1/targets/{target_id}/start HTTP/1.1
```

#### Example Request

```bash
curl \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -X PUT \
  -i \
  https://listener-app-services.teradata.com/v1/targets/758fbda4-accc-4f90-8f09-cc0a164c8c28/start
```

#### Example Response

```http
HTTP/1.1 200 OK
```

#### Response Codes

Code | Meaning
---- | -------
200  | Target was started successfully.
400  | Target settings may be invalid, such as incorrect mapping settings or permissions, which prevents the writer from starting.
401  | Authorization header was missing or could not be found.
404  | The resource was not found.







## Pause a Target

Only the owner or admin can pause a target, and doing so will deploy a new writer that will begin to pull data from the stream and write to the target.

#### Definition

```http
PUT https://listener-app-services.teradata.com/v1/targets/{target_id}/pause HTTP/1.1
```

#### Example Request

```bash
curl \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -X PUT \
  -i \
  https://listener-app-services.teradata.com/v1/targets/758fbda4-accc-4f90-8f09-cc0a164c8c28/pause
```

#### Example Response

```http
HTTP/1.1 200 OK
```

#### Response Codes

Code | Meaning
---- | -------
200  | Target was started successfully.
401  | Authorization header was missing or could not be found.
404  | The resource was not found.







## Get Target Status

A target's status is the current status of the writer, which can be one of four states: `started`, `starting`, `paused`, or `pausing`. Due to a writer taking some time to deploy, the `starting` and `pausing` statuses indicate the writer is changing state. In the case of a `paused` or `pausing` state the response will contain also a `reason`
field with a constant value to give more details about what put the writer in pause state. Indeed, a writer could have been paused due to a bad record encountered writing
a batch of data, due to a user request to stop a writer or update a target, or some internal error.

#### Definition

```http
GET https://listener-app-services.teradata.com/v1/targets/{target_id}/status HTTP/1.1
```

#### Example Request

```bash
curl \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -i \
  https://listener-app-services.teradata.com/v1/targets/758fbda4-accc-4f90-8f09-cc0a164c8c28/status
```

#### Example Response

```http
HTTP/1.1 200 OK
```
```json
{
  "status": "paused"
}
```

#### Example Response with reason

```http
HTTP/1.1 200 OK
```
```json
{
  "status": "started",
  "can_pause": false,
  "can_start": true,
  "error": "exception message",
  "reason": "BAD_RECORD"
}
```

#### Response Codes

Code | Meaning
---- | -------
200  | Status was returned successfully.
401  | Authorization header was missing or could not be found.
404  | The resource was not found.

#### Reason Values

Value          | Meaning
---------------| --------------
BAD_RECORD     | Bad record encountered writing data
USER_REQUESTED | User operation (e.g. Writer stopped by user)
RUNTIME_ERROR  | Internal error






## Validate a Target

You can validate a target's connection settings before trying to save it.

#### Definition

```http
POST https://listener-app-services.teradata.com/v1/targets/{target_id}/validate HTTP/1.1
```

#### Example Request

```bash
curl \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -X POST \
  -d '{
    "system_id": "f62bcf50-4cce-450d-be47-ca837918fdbd",
    "target_type": "teradata",
    "data_path": {
      "schema": "listener",
      "table": "demo"
    },
    "system_info": {
      "username": "dbc",
      "password": "dbc"
    }
  }'\
  -i \
  https://listener-app-services.teradata.com/v1/targets/validate
```

#### Example Response

```http
HTTP/1.1 200 OK
```

#### Response Codes

Code | Meaning
---- | -------
200  | Target details are valid.
400  | Validation failed due to missing or incorrect details.
401  | Authorization header was missing or could not be found.

## Retrieve Count of Dead Letter records

If a target has been configured to store bad records in a ```dead letter queue``` then you can retrieve the number of records held in the queue through the API.

This functionality is currently restricted to the owner of the target and the system admin user.

#### Definition

```http
GET https://listener-app-services.teradata.com/v1/targets/{target_id}/dead-letter-queue/count HTTP/1.1
```

#### Example Response

```http
HTTP/1.1 200 OK
```
```json
{
  "count" : 42
}
```

#### Response Codes

Code | Meaning
---- | -------
200  | Successfully retrieved record count
403  | User does not have permission to retrieve record count

## Retrieve Dead Letter records

If a target has been configured to store bad records in a ```dead letter queue```, you can retrieve the bad records through the API.

This function is currently restricted to the target owner and the system admin user.

#### Definition

```http
GET https://listener-app-services.teradata.com/v1/targets/{target_id}/dead-letter-queue/records HTTP/1.1
```

#### Example Response

```http
HTTP/1.1 200 OK
```
```json
[
  {
    "data": "{\"property\" : \"value\"",
    "date": "2017-05-18 11:23:48",
    "error_code": -7548,
    "error_reason": "Invalid JSON data: Expected something like whitespace or '}' or ']' or ',' between '\"' and the end of the string.",
    "uuid": "07e29abf-1e90-4327-9fd0-2769127fc69f"
  }
]
```

#### Response Codes

Code | Meaning
---- | -------
200  | Successfully retrieved records
403  | User does not have permission to retrieve records

## Clear Dead Letter Queue

If a target has been configured to store bad records in a ```dead letter queue```, you can clear the dead letter queue through the API.

This function is currently restricted to the target owner and the system admin user.

#### Definition

```http
DELETE https://listener-app-services.teradata.com/v1/targets/{target_id}/dead-letter-queue HTTP/1.1
```

#### Example Response

```http
HTTP/1.1 200 OK
```
```json
[
  {
    "message": "Topic cleared successfully",
    "success": true
  }
]
```

#### Response Codes

Code | Meaning
---- | -------
200  | Successfully cleared the dead letter queue
403  | User does not have permission to clear the dead letter queue
