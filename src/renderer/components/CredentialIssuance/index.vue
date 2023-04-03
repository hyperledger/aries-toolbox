<template>
  <el-row>
    <schema-list
      title="Schemas"
      editable="false"
      :list="schemas"
      @schema-send="publish_schema"
      @schema-get="get_schema"
      @schema-refresh="fetch_schemas"></schema-list>
    <cred-def-list
      title="Credential Definitions"
      :can_create="true"
      :retrievable="true"
      :list="cred_defs"
      @cred-def-refresh="fetch_cred_defs"></cred-def-list>
    <issued-cred-list
      title="Issued Credentials"
      v-bind:list="issued_credentials"
      v-bind:connections="active_connections"
      v-bind:cred_defs="issuer_cred_defs"
      @issue="issue_credential"
      @revoke="revoke_credential"
      @issue-cred-refresh="fetch_issued_credentials">
    </issued-cred-list>
  </el-row>
</template>

<script>
import message_bus from '../../message_bus.js';
import share from '../../share.js';
import SchemaList from './SchemaList.vue';
import CredDefList from './CredDefList.vue';
import IssuedCredList from './IssuedCredList.vue';

export const metadata = {
  menu: {
    label: 'Credential Issuance',
    icon: 'el-icon-document',
    group: 'Agent to Agent',
    priority: 50,
    required_protocols: [
      'did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-schemas/0.1',
      "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-issuer/0.1",
      "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-credential-definitions/0.1"
    ]
  }
};

export const shared = {
  data: {
    schemas: [],
    cred_defs: [],
    issued_credentials: []
  },
  computed: {
    issuer_cred_defs: function() {
      return this.cred_defs.filter(
        cred_def => {
          return cred_def.author === 'self' ||
            cred_def.cred_def_id.split(':', 2)[0] === this.public_did
        }
      );
    },
  },
  listeners: {
    "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-schemas/0.1/schema-list":
    (share, msg) => share.schemas = msg.results,
    "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-issuer/0.1/credentials-list":
    (share, msg) => share.issued_credentials = msg.results,
    "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-credential-definitions/0.1/credential-definition-list":
    (share, msg) => share.cred_defs = msg.results,
    "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-issuer/0.1/credential-issued":
    (share, msg) => share.fetch_issued_credentials()
  },
  methods: {
    fetch_schemas: ({send}) => {
      send({
        "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-schemas/0.1/schema-get-list"
      })
    },
    fetch_cred_defs: ({send}) => {
      send({
        "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-credential-definitions/0.1/credential-definition-get-list"
      })
    },
    fetch_issued_credentials: ({send}) => {
      send({
        "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-issuer/0.1/credentials-get-list",
      })
    },
  }
}

export default {
  name: 'credential-issuance',
  mixins: [
    message_bus({
      events: {
        "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-schemas/0.1/schema-id":
        (v, msg) => v.fetch_schemas(),
        "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-schemas/0.1/schema":
        (v, msg) => v.fetch_schemas(),
        "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-issuer/0.1/credential-exchange":
        (v, msg) => v.fetch_issued_credentials(),
      }
    }),
    share({
      use: [
        'active_connections',
        'schemas',
        'cred_defs',
        'issued_credentials',
        'issuer_cred_defs'
      ],
      actions: [
        'fetch_schemas',
        'fetch_cred_defs',
        'fetch_issued_credentials',
        'fetch_connections',
      ]
    })
  ],
  components: {
    SchemaList,
    CredDefList,
    IssuedCredList
  },
  created: async function() {
    this.$message_bus.$emit('entered_taa_required_module');
    await this.ready()
    this.fetch_schemas();
    this.fetch_cred_defs();
    this.fetch_issued_credentials();
    this.fetch_connections();
  },
  methods: {
    publish_schema: function(form) {
      let query_msg = {
        "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-schemas/0.1/send-schema",
        "schema_name": form.name,
        "schema_version": form.version,
        "attributes": form.attributes,
      };
      this.send_message(query_msg);
    },
    get_schema: function(schema_id) {
      let query_msg = {
        "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-schemas/0.1/schema-get",
        "schema_id": schema_id,
      };
      this.send_message(query_msg);
    },
    issue_credential: function(form) {
      let query_msg = {
        "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-issuer/0.1/send-credential",
        "connection_id": form.connection_id,
        "cred_def_id": form.cred_def_id,
        "comment": form.comment, //optional
        "credential_proposal": {
          "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/issue-credential/1.0/credential-preview",
          "attributes": form.attributes
        }
      };
      this.send_message(query_msg);
    },
    revoke_credential: function(form) {
      let query_msg = {
        "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-issuer/0.1/revoke-credential",
        "credential_exchange_id": form.credential_exchange_id,
      };
      this.send_message(query_msg);
    },
  }
}
</script>
