# Holder Admin Protocol

## Overview

**Protocol URI:** `https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1`

Define messages for credential holder admin protocols.

### Protocol Messages
- [credentials-get-list](#credentials-get-list)
- [credentials-list](#credentials-list)
- [credential-offer-accept](#credential-offer-accept)
- [credential-offer-received](#credential-offer-received)
- [credential-request-sent](#credential-request-sent)
- [credential-received](#credential-received)
- [credential-delete](#credential-delete)
- [send-credential-proposal](#send-credential-proposal)
- [credential-exchange](#credential-exchange)
- [presentations-get-list](#presentations-get-list)
- [presentations-list](#presentations-list)
- [presentation-request-approve](#presentation-request-approve)
- [presentation-request-reject](#presentation-request-reject)
- [presentation-get-matching-credentials](#presentation-get-matching-credentials)
- [presentation-matching-credentials](#presentation-matching-credentials)
- [send-presentation-proposal](#send-presentation-proposal)
- [presentation-exchange](#presentation-exchange)


## Message Definitions

### credentials-get-list

Credential list retrieval message.

Example:

```json
{
  "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/credentials-get-list",
  "@id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "~paginate": {
    "limit": 10,
    "offset": 20
  },
  "states": [
    "offer_received"
  ]
}
```

#### Fields

`@type` (String): Message type

`@id` (String; Optional): Message identifier

`~paginate` (Nested; Optional): Paginate decorator for messages querying for a paginated object.

`states` (List; Optional): Filter listed credentials by state.

### credentials-list

Credential list message.

Example:

```json
{
  "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/credentials-list",
  "@id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "results": [],
  "~page": {
    "count": 10,
    "offset": 20,
    "remaining": 15
  }
}
```

#### Fields

`@type` (String): Message type

`@id` (String; Optional): Message identifier

`results` (List): List of requested credentials

`~page` (Nested; Optional): Page decorator for messages containing a paginated object.

### credential-offer-accept

Credential offer accept message.

Example:

```json
{
  "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/credential-offer-accept",
  "@id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "credential_exchange_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```

#### Fields

`@type` (String): Message type

`@id` (String; Optional): Message identifier

`credential_exchange_id` (String): ID of the credential exchange to accept

### credential-offer-received

Credential offer received message.

Example:

```json
{
  "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/credential-offer-received",
  "@id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "state": "credential_acked",
  "created_at": "2021-04-13 14:55:53Z",
  "updated_at": "2021-04-13 14:55:53Z",
  "trace": null,
  "credential_exchange_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "connection_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "thread_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "parent_thread_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "initiator": "self",
  "role": "issuer",
  "credential_definition_id": "WgWxqztrNooG92RXvxSTWv:3:CL:20:tag",
  "schema_id": "WgWxqztrNooG92RXvxSTWv:2:schema_name:1.0",
  "credential_proposal_dict": null,
  "credential_offer_dict": null,
  "credential_offer": null,
  "credential_request": null,
  "credential_request_metadata": null,
  "credential_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "raw_credential": null,
  "credential": null,
  "auto_offer": false,
  "auto_issue": false,
  "auto_remove": false,
  "error_msg": "credential definition identifier is not set in proposal",
  "revoc_reg_id": null,
  "revocation_id": null
}
```

#### Fields

`@type` (String): Message type

`@id` (String; Optional): Message identifier

`state` (String; Optional): Issue-credential exchange state

`created_at` (String; Optional): Time of record creation

`updated_at` (String; Optional): Time of last record update

`trace` (Boolean; Optional): Record trace information, based on agent configuration

`credential_exchange_id` (String; Optional): Credential exchange identifier

`connection_id` (String; Optional): Connection identifier

`thread_id` (String; Optional): Thread identifier

`parent_thread_id` (String; Optional): Parent thread identifier

`initiator` (String; Optional): Issue-credential exchange initiator: self or external

`role` (String; Optional): Issue-credential exchange role: holder or issuer

`credential_definition_id` (String; Optional): Credential definition identifier

`schema_id` (String; Optional): Schema identifier

`credential_proposal_dict` (Dict; Optional): Serialized credential proposal message

`credential_offer_dict` (Dict; Optional): Serialized credential offer message

`credential_offer` (Dict; Optional): (Indy) credential offer

`credential_request` (Dict; Optional): (Indy) credential request

`credential_request_metadata` (Dict; Optional): (Indy) credential request metadata

`credential_id` (String; Optional): Credential identifier

`raw_credential` (Dict; Optional): Credential as received, prior to storage in holder wallet

`credential` (Dict; Optional): Credential as stored

`auto_offer` (Boolean; Optional): Holder choice to accept offer in this credential exchange

`auto_issue` (Boolean; Optional): Issuer choice to issue to request in this credential exchange

`auto_remove` (Boolean; Optional): Issuer choice to remove this credential exchange record when complete

`error_msg` (String; Optional): Error message

`revoc_reg_id` (String; Optional): Revocation registry identifier

`revocation_id` (String; Optional): Credential identifier within revocation registry

### credential-request-sent

Credential offer acceptance received and credential request sent.

Example:

```json
{
  "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/credential-request-sent",
  "@id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "state": "credential_acked",
  "created_at": "2021-04-13 14:55:53Z",
  "updated_at": "2021-04-13 14:55:53Z",
  "trace": null,
  "credential_exchange_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "connection_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "thread_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "parent_thread_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "initiator": "self",
  "role": "issuer",
  "credential_definition_id": "WgWxqztrNooG92RXvxSTWv:3:CL:20:tag",
  "schema_id": "WgWxqztrNooG92RXvxSTWv:2:schema_name:1.0",
  "credential_proposal_dict": null,
  "credential_offer_dict": null,
  "credential_offer": null,
  "credential_request": null,
  "credential_request_metadata": null,
  "credential_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "raw_credential": null,
  "credential": null,
  "auto_offer": false,
  "auto_issue": false,
  "auto_remove": false,
  "error_msg": "credential definition identifier is not set in proposal",
  "revoc_reg_id": null,
  "revocation_id": null
}
```

#### Fields

`@type` (String): Message type

`@id` (String; Optional): Message identifier

`state` (String; Optional): Issue-credential exchange state

`created_at` (String; Optional): Time of record creation

`updated_at` (String; Optional): Time of last record update

`trace` (Boolean; Optional): Record trace information, based on agent configuration

`credential_exchange_id` (String; Optional): Credential exchange identifier

`connection_id` (String; Optional): Connection identifier

`thread_id` (String; Optional): Thread identifier

`parent_thread_id` (String; Optional): Parent thread identifier

`initiator` (String; Optional): Issue-credential exchange initiator: self or external

`role` (String; Optional): Issue-credential exchange role: holder or issuer

`credential_definition_id` (String; Optional): Credential definition identifier

`schema_id` (String; Optional): Schema identifier

`credential_proposal_dict` (Dict; Optional): Serialized credential proposal message

`credential_offer_dict` (Dict; Optional): Serialized credential offer message

`credential_offer` (Dict; Optional): (Indy) credential offer

`credential_request` (Dict; Optional): (Indy) credential request

`credential_request_metadata` (Dict; Optional): (Indy) credential request metadata

`credential_id` (String; Optional): Credential identifier

`raw_credential` (Dict; Optional): Credential as received, prior to storage in holder wallet

`credential` (Dict; Optional): Credential as stored

`auto_offer` (Boolean; Optional): Holder choice to accept offer in this credential exchange

`auto_issue` (Boolean; Optional): Issuer choice to issue to request in this credential exchange

`auto_remove` (Boolean; Optional): Issuer choice to remove this credential exchange record when complete

`error_msg` (String; Optional): Error message

`revoc_reg_id` (String; Optional): Revocation registry identifier

`revocation_id` (String; Optional): Credential identifier within revocation registry

### credential-received

Credential received notification message.

Example:

```json
{
  "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/credential-received",
  "@id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "state": "credential_acked",
  "created_at": "2021-04-13 14:55:53Z",
  "updated_at": "2021-04-13 14:55:53Z",
  "trace": null,
  "credential_exchange_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "connection_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "thread_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "parent_thread_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "initiator": "self",
  "role": "issuer",
  "credential_definition_id": "WgWxqztrNooG92RXvxSTWv:3:CL:20:tag",
  "schema_id": "WgWxqztrNooG92RXvxSTWv:2:schema_name:1.0",
  "credential_proposal_dict": null,
  "credential_offer_dict": null,
  "credential_offer": null,
  "credential_request": null,
  "credential_request_metadata": null,
  "credential_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "raw_credential": null,
  "credential": null,
  "auto_offer": false,
  "auto_issue": false,
  "auto_remove": false,
  "error_msg": "credential definition identifier is not set in proposal",
  "revoc_reg_id": null,
  "revocation_id": null
}
```

#### Fields

`@type` (String): Message type

`@id` (String; Optional): Message identifier

`state` (String; Optional): Issue-credential exchange state

`created_at` (String; Optional): Time of record creation

`updated_at` (String; Optional): Time of last record update

`trace` (Boolean; Optional): Record trace information, based on agent configuration

`credential_exchange_id` (String; Optional): Credential exchange identifier

`connection_id` (String; Optional): Connection identifier

`thread_id` (String; Optional): Thread identifier

`parent_thread_id` (String; Optional): Parent thread identifier

`initiator` (String; Optional): Issue-credential exchange initiator: self or external

`role` (String; Optional): Issue-credential exchange role: holder or issuer

`credential_definition_id` (String; Optional): Credential definition identifier

`schema_id` (String; Optional): Schema identifier

`credential_proposal_dict` (Dict; Optional): Serialized credential proposal message

`credential_offer_dict` (Dict; Optional): Serialized credential offer message

`credential_offer` (Dict; Optional): (Indy) credential offer

`credential_request` (Dict; Optional): (Indy) credential request

`credential_request_metadata` (Dict; Optional): (Indy) credential request metadata

`credential_id` (String; Optional): Credential identifier

`raw_credential` (Dict; Optional): Credential as received, prior to storage in holder wallet

`credential` (Dict; Optional): Credential as stored

`auto_offer` (Boolean; Optional): Holder choice to accept offer in this credential exchange

`auto_issue` (Boolean; Optional): Issuer choice to issue to request in this credential exchange

`auto_remove` (Boolean; Optional): Issuer choice to remove this credential exchange record when complete

`error_msg` (String; Optional): Error message

`revoc_reg_id` (String; Optional): Revocation registry identifier

`revocation_id` (String; Optional): Credential identifier within revocation registry

### credential-delete

Delete existing credential exchange.

Example:

```json
{
  "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-holder/0.1/credential-delete",
  "credential_exchange_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```

#### Fields

`@type` (String): Message type

`credential_exchange_id` (String; Optional): Credential exchange identifier

### send-credential-proposal

Send Credential Proposal Message.

Example:

```json
{
  "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/send-credential-proposal",
  "@id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "trace": false,
  "connection_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "cred_def_id": "WgWxqztrNooG92RXvxSTWv:3:CL:20:tag",
  "schema_id": "WgWxqztrNooG92RXvxSTWv:2:schema_name:1.0",
  "schema_issuer_did": "WgWxqztrNooG92RXvxSTWv",
  "schema_name": "preferences",
  "schema_version": "1.0",
  "issuer_did": "WgWxqztrNooG92RXvxSTWv",
  "auto_remove": null,
  "comment": null,
  "credential_proposal": {
    "@type": "issue-credential/1.0/credential-preview",
    "attributes": {
      "name": "favourite_drink",
      "mime-type": "image/jpeg",
      "value": "martini"
    }
  }
}
```

#### Fields

`@type` (String): Message type

`@id` (String; Optional): Message identifier

`trace` (Boolean; Optional): Whether to trace event (default false)

`connection_id` (UUID): Connection identifier

`cred_def_id` (String; Optional): Credential definition identifier

`schema_id` (String; Optional): Schema identifier

`schema_issuer_did` (String; Optional): Schema issuer DID

`schema_name` (String; Optional): Schema name

`schema_version` (String; Optional): Schema version

`issuer_did` (String; Optional): Credential issuer DID

`auto_remove` (Boolean; Optional): Whether to remove the credential exchange record on completion (overrides --preserve-exchange-records configuration setting)

`comment` (String; Optional): Human-readable comment

`credential_proposal` (Nested): Class representing a credential preview inner object.

### credential-exchange

Credential exchange message.

Example:

```json
{
  "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/credential-exchange",
  "@id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "state": "credential_acked",
  "created_at": "2021-04-13 14:55:53Z",
  "updated_at": "2021-04-13 14:55:53Z",
  "trace": null,
  "credential_exchange_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "connection_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "thread_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "parent_thread_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "initiator": "self",
  "role": "issuer",
  "credential_definition_id": "WgWxqztrNooG92RXvxSTWv:3:CL:20:tag",
  "schema_id": "WgWxqztrNooG92RXvxSTWv:2:schema_name:1.0",
  "credential_proposal_dict": null,
  "credential_offer_dict": null,
  "credential_offer": null,
  "credential_request": null,
  "credential_request_metadata": null,
  "credential_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "raw_credential": null,
  "credential": null,
  "auto_offer": false,
  "auto_issue": false,
  "auto_remove": false,
  "error_msg": "credential definition identifier is not set in proposal",
  "revoc_reg_id": null,
  "revocation_id": null
}
```

#### Fields

`@type` (String): Message type

`@id` (String; Optional): Message identifier

`state` (String; Optional): Issue-credential exchange state

`created_at` (String; Optional): Time of record creation

`updated_at` (String; Optional): Time of last record update

`trace` (Boolean; Optional): Record trace information, based on agent configuration

`credential_exchange_id` (String; Optional): Credential exchange identifier

`connection_id` (String; Optional): Connection identifier

`thread_id` (String; Optional): Thread identifier

`parent_thread_id` (String; Optional): Parent thread identifier

`initiator` (String; Optional): Issue-credential exchange initiator: self or external

`role` (String; Optional): Issue-credential exchange role: holder or issuer

`credential_definition_id` (String; Optional): Credential definition identifier

`schema_id` (String; Optional): Schema identifier

`credential_proposal_dict` (Dict; Optional): Serialized credential proposal message

`credential_offer_dict` (Dict; Optional): Serialized credential offer message

`credential_offer` (Dict; Optional): (Indy) credential offer

`credential_request` (Dict; Optional): (Indy) credential request

`credential_request_metadata` (Dict; Optional): (Indy) credential request metadata

`credential_id` (String; Optional): Credential identifier

`raw_credential` (Dict; Optional): Credential as received, prior to storage in holder wallet

`credential` (Dict; Optional): Credential as stored

`auto_offer` (Boolean; Optional): Holder choice to accept offer in this credential exchange

`auto_issue` (Boolean; Optional): Issuer choice to issue to request in this credential exchange

`auto_remove` (Boolean; Optional): Issuer choice to remove this credential exchange record when complete

`error_msg` (String; Optional): Error message

`revoc_reg_id` (String; Optional): Revocation registry identifier

`revocation_id` (String; Optional): Credential identifier within revocation registry

### presentations-get-list

Presentation get list message.

Example:

```json
{
  "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/presentations-get-list",
  "@id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "connection_id": null,
  "~paginate": {
    "limit": 10,
    "offset": 20
  }
}
```

#### Fields

`@type` (String): Message type

`@id` (String; Optional): Message identifier

`connection_id` (String; Optional): Filter presentations by connection_id

`~paginate` (Nested; Optional): Pagination decorator.

### presentations-list

Presentation get list response message.

Example:

```json
{
  "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/presentations-list",
  "@id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "results": null,
  "~page": {
    "count": 10,
    "offset": 20,
    "remaining": 15
  }
}
```

#### Fields

`@type` (String): Message type

`@id` (String; Optional): Message identifier

`results` (List; Optional): Retrieved presentations.

`~page` (Nested; Optional): Pagination decorator.

### presentation-request-approve

Approve presentation request.

Example:

```json
{
  "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/presentation-request-approve",
  "@id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "presentation_exchange_id": null,
  "self_attested_attributes": null,
  "requested_attributes": null,
  "requested_predicates": null,
  "comment": "Nothing to see here."
}
```

#### Fields

`@type` (String): Message type

`@id` (String; Optional): Message identifier

`presentation_exchange_id` (String): 

`self_attested_attributes` (Dict): Self-attested attributes to build into proof

`requested_attributes` (Dict): Nested object mapping proof request attribute referents to requested-attribute specifiers

`requested_predicates` (Dict): Nested object mapping proof request predicate referents to requested-predicate specifiers

`comment` (String; Optional): Optional comment.

### presentation-request-reject

Reject presentation request.

Example:

```json
{
  "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-holder/0.1/presentation-request-reject",
  "@id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "presentation_exchange_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "message_description": "Description of problem",
}
```

#### Fields

`@type` (String): Message type

`@id` (String; Optional): Message identifier

`presentation_exchange_id` (String): Presentation to reject.

`message_description` (String; Optional): Description of cause of rejection.  

### presentation-get-matching-credentials

Retrieve matching credentials for a presentation request.

Example:

```json
{
  "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/presentation-get-matching-credentials",
  "@id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "presentation_exchange_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "~paginate": {
    "limit": 10,
    "offset": 20
  }
}
```

#### Fields

`@type` (String): Message type

`@id` (String; Optional): Message identifier

`presentation_exchange_id` (String): Presentation to match credentials to.

`~paginate` (Nested; Optional): Pagination decorator.

### presentation-matching-credentials

Presentation Matching Credentials

Example:

```json
{
  "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/presentation-matching-credentials",
  "@id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "presentation_exchange_id": null,
  "matching_credentials": {
    "cred_info": {
      "referent": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "attrs": null
    },
    "schema_id": "WgWxqztrNooG92RXvxSTWv:2:schema_name:1.0",
    "cred_def_id": "WgWxqztrNooG92RXvxSTWv:3:CL:20:tag",
    "rev_reg_id": "WgWxqztrNooG92RXvxSTWv:4:WgWxqztrNooG92RXvxSTWv:3:CL:20:tag:CL_ACCUM:0",
    "cred_rev": "12345",
    "interval": {
      "to": 1618516861
    },
    "presentation_referents": null
  },
  "page": {
    "count": 10,
    "offset": 20,
    "remaining": 15
  }
}
```

#### Fields

`@type` (String): Message type

`@id` (String; Optional): Message identifier

`presentation_exchange_id` (String): Exchange ID for matched credentials.

`matching_credentials` (Nested; Optional): Matched credentials.

`page` (Nested; Optional): Pagination info for matched credentials.

### send-presentation-proposal

Presentation proposal message.

Example:

```json
{
  "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/send-presentation-proposal",
  "@id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "trace": false,
  "connection_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "comment": null,
  "presentation_proposal": {
    "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/present-proof/1.0/presentation-preview",
    "attributes": {
      "name": "favourite_drink",
      "cred_def_id": "WgWxqztrNooG92RXvxSTWv:3:CL:20:tag",
      "mime-type": "image/jpeg",
      "value": "martini",
      "referent": "0"
    },
    "predicates": {
      "name": "high_score",
      "cred_def_id": "WgWxqztrNooG92RXvxSTWv:3:CL:20:tag",
      "predicate": ">=",
      "threshold": null
    }
  },
  "auto_present": null
}
```

#### Fields

`@type` (String): Message type

`@id` (String; Optional): Message identifier

`trace` (Boolean; Optional): Whether to trace event (default false)

`connection_id` (UUID): Connection identifier

`comment` (String; Optional): Human-readable comment

`presentation_proposal` (Nested): Class representing presentation preview.

`auto_present` (Boolean; Optional): Whether to respond automatically to presentation requests, building and presenting requested proof

### presentation-exchange

Presentation Exchange message.

Example:

```json
{
  "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/presentation-exchange",
  "@id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "state": "verified",
  "created_at": "2021-04-13 14:55:53Z",
  "updated_at": "2021-04-13 14:55:53Z",
  "trace": null,
  "presentation_exchange_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "connection_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "thread_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "initiator": "self",
  "role": "prover",
  "presentation_proposal_dict": null,
  "presentation_request": null,
  "presentation_request_dict": null,
  "presentation": null,
  "verified": "true",
  "auto_present": false,
  "error_msg": "Invalid structure"
}
```

#### Fields

`@type` (String): Message type

`@id` (String; Optional): Message identifier

`state` (String; Optional): Present-proof exchange state

`created_at` (String; Optional): Time of record creation

`updated_at` (String; Optional): Time of last record update

`trace` (Boolean; Optional): Record trace information, based on agent configuration

`presentation_exchange_id` (String; Optional): Presentation exchange identifier

`connection_id` (String; Optional): Connection identifier

`thread_id` (String; Optional): Thread identifier

`initiator` (String; Optional): Present-proof exchange initiator: self or external

`role` (String; Optional): Present-proof exchange role: prover or verifier

`presentation_proposal_dict` (Dict; Optional): Serialized presentation proposal message

`presentation_request` (Dict; Optional): (Indy) presentation request (also known as proof request)

`presentation_request_dict` (Dict; Optional): Serialized presentation request message

`presentation` (Dict; Optional): (Indy) presentation (also known as proof)

`verified` (String; Optional): Whether presentation is verified: true or false

`auto_present` (Boolean; Optional): Prover choice to auto-present proof as verifier requests

`error_msg` (String; Optional): Error message
