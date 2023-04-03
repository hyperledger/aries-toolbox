<template >
  <el-row>
    <presentation
      title="Presentations"
      v-bind:presentations= "holder_presentations"
      v-bind:connections = "active_connections"
      v-bind:cred_defs = "cred_defs"
      v-bind:connection_details = "id_to_connection"
      @presentation-refresh = "fetch_holder_presentations"
      @send-presentation-proposal= "sendPresentationProposal"
      ></presentation>
  </el-row>
</template>

<script>
import VueJsonPretty from 'vue-json-pretty';
const { clipboard } = require('electron');
import Presentation from './Presentation.vue';
import message_bus from '@/message_bus.js';
import share from '@/share.js';

export const metadata = {
  menu: {
    label: 'Presentations',
    icon: 'el-icon-document-checked',
    group: 'Agent to Agent',
    priority: 90,
    required_protocols: [
      {'https' : 'https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1'},
      {'did:sov' : 'did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-holder/0.1'}
    ]
  }
};

export const shared = {
  data: {
    holder_presentations: [],
  },
  listeners: {
    'did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-holder/0.1/presentations-list':
    (share, msg) => share.holder_presentations = msg.results,
    'https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/presentations-list':
    (share, msg) => share.holder_presentations = msg.results,
    'did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-holder/0.1/presentation-request-received':
    (share, msg) => {
        share.holder_presentations.push(msg.raw_repr)
    },
    'https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/presentation-request-received':
    (share, msg) => {
        share.holder_presentations.push(msg.raw_repr)
    },
  },
  methods: {
    fetch_holder_presentations: ({send}) => {
      send({
        "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/presentations-get-list",
      })
    }
  }
}

export default {
  name: 'presentations',
  components: {
    VueJsonPretty,
    Presentation,
  },
  mixins: [
    message_bus(),
    share({
      use: [
        'active_connections', 
        'cred_defs',
        'holder_presentations',
        'id_to_connection'
      ],
      actions: ['fetch_holder_presentations']
    })
  ],
  created: async function() {
    await this.ready();
    this.fetch_holder_presentations();
  },
  methods: {
    async sendPresentationProposal(form){
      let query_msg = {
        "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-holder/0.1/send-presentation-proposal",
        "connection_id": form.connection_id,
        "comment": form.comment,
        "auto_present": form.auto_present , //optional, default to false
        "presentation_proposal": {
          "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/present-proof/1.0/presentation-preview",
          /**
           * name 
           * cred_def_id //optional
           * mime_type //optional
           * value //optional
           * */
          attributes: form.attributes.map(attribute => {
            return {
              name: attribute.name,
              cred_def_id: attribute.cred_def.cred_def_id,
              value: attribute.value
            };
          }),
          /**
           * name
           * cred_def_id
           * predicate
           * threshold
           */
          predicates: form.predicates.map(predicate => {
            return {
              name: predicate.name,
              cred_def_id: predicate.cred_def.cred_def_id,
              predicate: predicate.predicate,
              threshold: parseInt(predicate.threshold)
            };
          })
        },
      };
        this.send_message(query_msg);
    },
  }
}
</script>
