<template>
  <el-row>
    <cred-def-list
      title="Retrieved Credential Definitions"
      v-bind:retrievable="true"
      v-bind:can_create="false"
      v-bind:list="proposal_cred_defs"></cred-def-list>
    <my-credentials-list
      title="Credentials"
      editable="false"
      v-bind:list="holder_credentials"
      v-bind:cred_defs="proposal_cred_defs"
      v-bind:connections="active_connections"
      @cred-refresh="fetch_holder_credentials"
      @propose="send_proposal"></my-credentials-list>
  </el-row>
</template>

<script>
import CredDefList from '../CredentialIssuance/CredDefList.vue';
import MyCredentialsList from './MyCredentialsList.vue';
import message_bus from '../../message_bus.js';
import share from '../../share.js';

export const metadata = {
  menu: {
    label: 'My Credentials',
    icon: 'el-icon-bank-card',
    group: 'Agent to Agent',
    priority: 60,
    required_protocols: [
      {'https' : "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1"},
      {'did:sov' : 'did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-holder/0.1'}
    ]
  }
};

export const shared = {
  data: {
    holder_credentials: [],
  },
  computed: {
    proposal_cred_defs: function() {
      return this.cred_defs.filter(
        cred_def => {
          return cred_def.author !== 'self' ||
            cred_def.cred_def_id.split(':', 2)[0] !== this.public_did
        }
      );
    },
  },
  listeners: {
    "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-holder/0.1/credentials-list":
    (share, msg) => share.holder_credentials = msg.results,
    "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/credentials-list":
    (share, msg) => share.holder_credentials = msg.results,
    "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-holder/0.1/credential-offer-received":
    (share, msg) => share.holder_credentials.push(msg.raw_repr),
    "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/credential-offer-received":
    (share, msg) => share.holder_credentials.push(msg.raw_repr),
    "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-holder/0.1/credential-received":
    (share, msg) => share.fetch_holder_credentials(),
    "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/credential-received":
    (share, msg) => share.fetch_holder_credentials()
  },
  methods: {
    fetch_holder_credentials: ({send}) => {
      send({
        "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/credentials-get-list",
      })
    }
  }
}

export default {
  name: 'my-credentials',
  components: {
    CredDefList,
    MyCredentialsList
  },
  mixins: [
    message_bus({
      events: {
        "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/credential-exchange":
        (v, msg) => setTimeout(v.fetch_holder_credentials, 4500),
        "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-holder/0.1/credential-exchange":
        (v, msg) => setTimeout(v.fetch_holder_credentials, 4500),
        'my-credentials': (v) => v.fetch_holder_credentials()
      }
    }),
    share({
      use: [
        'holder_credentials',
        'proposal_cred_defs',
        'active_connections'
      ],
      actions: ['fetch_holder_credentials']
    })
  ],
  methods: {
    send_proposal: function(form) {
      let query_msg = {
        "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/send-credential-proposal",
        "connection_id": form.connection_id,
        "credential_definition_id": form.cred_def_id,
        "comment": form.comment, //optional
        "credential_proposal": {
          "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/issue-credential/1.0/credential-preview",
          "attributes": form.attributes
        }
      };
      this.send_message(query_msg);
    }
  },
  created: function() {
    this.fetch_holder_credentials();
  }
}
</script>
