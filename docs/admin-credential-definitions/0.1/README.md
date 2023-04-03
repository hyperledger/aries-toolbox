# Credential Definition Admin Protocol

## Overview
**Protocol URI:** `did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-credential-definitions/0.1`


#### Protocol Messages
* [send-credential-definition](#send-credential-definition)
* [credential-definition-id](#credential-definition-id)
* [credential-definition-get](#credential-definition-get)
* [credential-definition](#credential-definition)
* [credential-definition-get-list](#credential-definition-get-list)
* [credential-definition-list](#credential-definition-list)

## Message Definitions

### send-credential-definition
Create a credential definition based on a schema.
```
{
    "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-credential-definitions/0.1/send-credential-definition",
    "schema_id": "5P4NvotLsVeeGd9UxUtMp6:2:Alice's Test Schema:1.0",
    "@id": "4e92857c-2b52-4e15-8e12-be695cb45ca5",
    "~transport": {
        "return_route": "all"
    }
}
```


### credential-definition-id
Response to a `send-credential-definition` message.
```
{
    "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-credential-definitions/0.1/credential-definition-id",
    "@id": "795932f4-dd26-4a9c-8b7e-e8c2302c9683",
    "~thread": {
        "thid": "4e92857c-2b52-4e15-8e12-be695cb45ca5"
    },
    "cred_def_id": "K6fuxNHhNrR44RfkF5jRp7:3:CL:232915:Alice's Test Schema_1.0"
}
```


### credential-definition-get
Retrieve a pre-existing credential definition.

```
{
    "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-credential-definitions/0.1/credential-definition-get",
    "cred_def_id": "K6fuxNHhNrR44RfkF5jRp7:3:CL:232915:Alice's Test Schema_1.0",
    "@id": "9c107592-1786-469a-9a24-eade00d5629f",
    "~transport": {
        "return_route": "all"
    }
}
```


### credential-definition
Respond message to a `credential-definition-get` message.
```
{
    "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-credential-definitions/0.1/credential-definition",
    "@id": "0299d15e-9fb1-4e8d-884e-c1ed8e6e7c87",
    "~thread": {
        "thid": "9c107592-1786-469a-9a24-eade00d5629f"
    },
    "updated_at": "2021-07-15 16:17:26.446239Z",
    "schema_id": "5P4NvotLsVeeGd9UxUtMp6:2:Alice's Test Schema:1.0",
    "state": "written",
    "attributes": [
        "test_attr_0",
        "test_attr_1",
        "test_attr_2"
    ],
    "cred_def_id": "K6fuxNHhNrR44RfkF5jRp7:3:CL:232915:Alice's Test Schema_1.0",
    "author": "other",
    "created_at": "2021-07-15 16:17:26.446239Z"
}
```


### credential-definition-get-list
Retrieve the list of credential definitions.

```
{
    "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-credential-definitions/0.1/credential-definition-get-list",
    "@id": "33d0c436-558e-47b2-9a17-f66be8f869ca",
    "~transport": {
        "return_route": "all"
    }
}
```

### credential-definition-list
Respond to a `credential-definition-get-list` message.

```
{
    "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-credential-definitions/0.1/credential-definition-list",
    "@id": "9d070822-821f-4f5c-821f-c92242dc4818",
    "~thread": {
        "thid": "33d0c436-558e-47b2-9a17-f66be8f869ca"
    },
    "results": [
        {
            "author": "self",
            "created_at": "2021-07-15 16:00:10.829578Z",
            "state": "written",
            "updated_at": "2021-07-15 16:00:10.829578Z",
            "attributes": [
                "test_attr_0",
                "test_attr_1",
                "test_attr_2"
            ],
            "cred_def_id": "K6fuxNHhNrR44RfkF5jRp7:3:CL:232915:Alice's Test Schema_1.0",
            "schema_id": "5P4NvotLsVeeGd9UxUtMp6:2:Alice's Test Schema:1.0"
        }
    ]
}
```