<template >
  <el-row>
    <verification
      title="Verification"
      v-bind:list="issuer_presentations"
      v-bind:connections="active_connections"
      v-bind:cred_defs="cred_defs"
      @verification-refresh="fetch_issuer_presentations"
      @presentation-request="presentation_request"></verification>
  </el-row>
</template>

<script>
import VueJsonPretty from 'vue-json-pretty';
import Verification from './Verification.vue';
import message_bus from '@/message_bus.js';
import share from '@/share.js';


export const metadata = {
  menu: {
    label: 'Verification',
    icon: 'el-icon-s-claim',
    group: 'Agent to Agent',
    priority: 100,
    required_protocols: [
      {'https' : 'https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-issuer/0.1'},
      {'did:sov' : 'did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-issuer/0.1'}
    ]
  }
};

export const shared = {
  data: {
    issuer_presentations: [],
  },
  listeners: {
    "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-issuer/0.1/presentations-list":
    (share, msg) => share.issuer_presentations = msg.results,
    "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-issuer/0.1/presentations-list":
    (share, msg) => share.issuer_presentations = msg.results,
    "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-issuer/0.1/presentation-received":
    (share, msg) => share.fetch_issuer_presentations(),
    "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-issuer/0.1/presentation-received":
    (share, msg) => share.fetch_issuer_presentations()
  },
  methods: {
    fetch_issuer_presentations: ({send}) => {
      send({
        "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-issuer/0.1/presentations-get-list"
      })
    }
  }
}

export default {
  name: 'verifications',
  components: {
    VueJsonPretty,
    Verification,
  },
  mixins: [
    message_bus({
      events: {
        "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-issuer/0.1/presentation-exchange":
        (v, msg) => setTimeout(v.fetch_issuer_presentations, 4500),
        "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-issuer/0.1/presentation-exchange":
        (v, msg) => setTimeout(v.fetch_issuer_presentations, 4500),
        "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-issuer/0.1/request-presentation":
        (v, msg) => setTimeout(v.fetch_issuer_presentations, 4500),
        "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-issuer/0.1/request-presentation":
        (v, msg) => setTimeout(v.fetch_issuer_presentations, 4500),
      }
    }),
    share({
      use: [
        'issuer_presentations',
        'active_connections',
        'cred_defs',
      ],
      actions: ['fetch_issuer_presentations']
    })
  ],
  created: async function() {
    this.fetch_issuer_presentations();
  },
  methods: {
    async presentation_request(form){
      // response comes back in admin-issuer/0.1/presentation-exchange
      let query_msg = {
        "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-issuer/0.1/request-presentation",
        connection_id: form.connection_id,
        comment: form.comment,
        proof_request: {
          name: form.name,
          version: "1.0",
          requested_attributes: form.attributes.reduce((acc, attribute) => {
            let transmuted_attr = {
              name: attribute.name,
              restrictions: []
            };
            if (attribute.restrictions.cred_def) {
              transmuted_attr.restrictions.push({});
            }
            if (attribute.restrictions.cred_def) {
              transmuted_attr.restrictions[0].cred_def_id = attribute.restrictions.cred_def.cred_def_id;
            }
            if (attribute.restrictions.trusted_issuer) {
              transmuted_attr.restrictions[0].issuer_did = attribute.restrictions.trusted_issuer;
            }
            acc[attribute.name] = transmuted_attr;
            return acc;
          }, {}),
          requested_predicates: form.predicates.reduce((acc, predicate) => {
            let transmuted_pred = {
              name: predicate.name,
              p_type: predicate.p_type,
              p_value: parseInt(predicate.threshold),
              restrictions: []
            };
            if (predicate.restrictions.cred_def) {
              transmuted_pred.restrictions.push({});
            }
            if (predicate.restrictions.cred_def) {
              transmuted_pred.restrictions[0].cred_def_id = predicate.restrictions.cred_def.cred_def_id;
            }
            if (predicate.restrictions.trusted_issuer) {
              transmuted_pred.restrictions[0].issuer_did = predicate.restrictions.trusted_issuer;
            }
            acc[predicate.name] = transmuted_pred;
            return acc;
          }, {}),
        },
      };
      if (form.non_revoked) {
        query_msg.proof_request.non_revoked = {"to": new Date().getTime()};
      }
      this.send_message(query_msg);
    },
  },
}
</script>
