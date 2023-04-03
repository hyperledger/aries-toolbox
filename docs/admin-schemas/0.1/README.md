# Schemas Admin Protocol

## Overview
**Protocol URI:** `https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-schemas/0.1`


#### Protocol Messages
* [send-schema](#send-schema)
* [schema-id](#schema-id)
* [schema-get](#schema-id)
* [schema](#schema)
* [schema-get-list](#schema-get-list)
* [schema-list](#schema-list)

## Message Definitions

### Send schema
Create a schema by inputting a name, version number, and attributes.

```
{
    "@type":"https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-schemas/0.1/send-schema",
    "schema_name": "Alice's Test Schema",
    "schema_version": "1.0",
    "attributes": [
        "test_attr_0",
        "test_attr_1",
        "test_attr_2"
    ],
    "@id": "ad285eef-a5e4-4cea-9a40-12f3294d1826",
    "~transport": {
    "return_route": "all"
    }
}
```

### Schema ID
Response to a `send-schema` message.
```
{
    "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-schemas/0.1/schema-id",
    "@id": "c3091caa-ebcd-4d2c-a2fc-636893ee6f43",
    "~thread": {
        "thid": "ad285eef-a5e4-4cea-9a40-12f3294d1826"
    },
    "schema_id": "UjF64u8jDEEuRve7PKQGUo:2:Alice's Test Schema:1.0"
}
```


### Schema get
Retrieve a pre-existing schema using the schema ID.
```
{
    "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-schemas/0.1/schema-get",
    "schema_id": "UjF64u8jDEEuRve7PKQGUo:2:Alice's Test Schema:1.0",
    "@id": "32a22f71-eed6-4113-8732-c791302da893",
    "~transport": {
        "return_route": "all"
    }
}
```

### Schema
Response to a `schema-get` message.
```
{
    "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-schemas/0.1/schema",
    "@id": "7bc2adbc-e1f2-43be-9d8c-0890fb4e7180",
    "~thread": {
        "thid": "32a22f71-eed6-4113-8732-c791302da893"
    },
    "schema_version": "1.0",
    "updated_at": "2021-07-12 20:55:44.464312Z",
    "state": "written",
    "attributes": [
        "test_attr_0",
        "test_attr_2",
        "test_attr_1"
    ],
    "schema_id": "UjF64u8jDEEuRve7PKQGUo:2:Alice's Test Schema:1.0",
    "created_at": "2021-07-12 20:55:44.464312Z",
    "author": "other",
    "schema_name": "Alice's Test Schema"
}
```



### Schema get list
Retrieve the list of schemas.
```
{
    "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-schemas/0.1/schema-get-list",
    "@id": "f8639a9c-b1aa-4df5-ad93-bb91d94ebb1c",
    "~transport": {
        "return_route": "all"
    }
}
```

### Schema list
Response message to `schema-get-list`.
```
{
    "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-schemas/0.1/schema-list",
    "@id": "a94ef367-1892-435b-8522-dc38889dfccb",
    "~thread": {
        "thid": "f8639a9c-b1aa-4df5-ad93-bb91d94ebb1c"
    },
    "results": [
        {
            "schema_name": "Alice's Test Schema",
            "state": "written",
            "schema_id": "UjF64u8jDEEuRve7PKQGUo:2:Alice's Test Schema:1.0",
            "schema_version": "1.0",
            "attributes": [
                "test_attr_0",
                "test_attr_1",
                "test_attr_2"
            ],
            "updated_at": "2021-07-12 20:45:43.493398Z",
            "author": "self",
            "created_at": "2021-07-12 20:45:43.493398Z"
        }
    ]
}
```
