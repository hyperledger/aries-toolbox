<template >

  <el-row>
    <nav class="navbar navbar-expand-lg navbar-light bg-light">
      <a class="navbar-brand" href="#">Static Connections</a>
      <el-button
        type="primary"
        icon="el-icon-refresh"
        @click="fetch_static_connections"></el-button>
    </nav>
    <el-collapse v-model="expanded_items">
      <ul class="list">
        <el-collapse-item
          v-for="c in static_connections"
          v-bind:title="get_name(c)"
          :name="get_name(c)"
          :key="c.connection_id">
          <el-row :key="c.connection_id">
            <p> <b>My DID:</b> {{c.my_info.did}}
            <el-button
              type="info"
              size="mini" circle
              icon="el-icon-copy-document"
              @click="copyItem(c.my_info.did, 'DID')"></el-button>
            </p>
            <p><b>My VK:</b> {{c.my_info.vk}}
            <el-button
              type="info"
              icon="el-icon-copy-document"
              size="mini" circle
              @click="copyItem(c.my_info.vk, 'VK')"></el-button>
            </p>
            <p><b>My Endpoint:</b> {{c.my_info.endpoint}}
            <el-button
              type="info"
              icon="el-icon-copy-document"
              size="mini" circle
              @click="copyItem(c.my_info.endpoint, 'Endpoint')"></el-button>
            </p>
            <p><b>Remote DID:</b> {{c.their_info.did}}</p>
            <p><b>Remote VK:</b> {{c.their_info.vk}}</p>
            <div>
              <vue-json-pretty
                :deep=1
                :data="c">
              </vue-json-pretty>
            </div>
          </el-row>
        </el-collapse-item>
      </ul>
    </el-collapse>

    <p>Add Static Connection:</p>
  <el-form :inline="false" label-width="120px">
    <link rel="shortcut icon" href="/static"/>
    <el-form-item label="Label:">
      <el-input v-model="static_agent_form.label" style="width:200px;"> </el-input>
      <i>Static agent label.</i>
    </el-form-item>
    <el-form-item label="Role:">
      <el-input v-model="static_agent_form.role" style="width:200px;"> </el-input>
      <i>Static agent role.</i>
    </el-form-item>
    <el-form-item label="Static DID:">
      <el-input v-model="static_agent_form.static_did" style="width:200px;"> </el-input>
      <i>Static DID.</i>
    </el-form-item>
    <el-form-item label="Static Key:">
      <el-input v-model="static_agent_form.static_key" style="width:200px;"> </el-input>
      <i>Static key.</i>
    </el-form-item>
    <el-form-item label="Endpoint:">
      <el-input v-model="static_agent_form.static_endpoint" style="width:200px;"> </el-input>
      <i>Static endpoint.</i>
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="add">Add Static Agent</el-button>
    </el-form-item>
  </el-form>
  </el-row>
</template>

<script>
import VueJsonPretty from 'vue-json-pretty';
const { clipboard } = require('electron');
import share from '@/share.js';
import message_bus from '@/message_bus.js';

export const metadata = {
  menu: {
    label: 'Static Connections',
    icon: 'el-icon-box',
    group: 'Agent to Agent',
    priority: 40,
    required_protocols: [
      {'https' : 'https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-static-connections/0.1'},
      {'did:sov' : 'did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-static-connections/0.1'}
    ]
  }
};

export const shared = {
  data: {
    static_connections: []
  },
  listeners: {
    "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-static-connections/0.1/static-connection-list": (share, msg) => {
      share.static_connections = msg.results;
    },
    "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-static-connections/0.1/static-connection-list": (share, msg) => {
      share.static_connections = msg.results;
    },
    "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-static-connections/0.1/static-connection-info": (share, msg) => {
      share.fetch_static_connections();
    },
    "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-static-connections/0.1/static-connection-info": (share, msg) => {
      share.fetch_static_connections();
    }
  },
  methods: {
    fetch_static_connections: ({send}) => {
      send({
        "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-static-connections/0.1/static-connection-get-list"
      });
    }
  }
}

export default {
  name: 'static-connections',
  mixins: [
    message_bus(),
    share({
      use: ['static_connections'],
      actions: ['fetch_static_connections']
    })
  ],
  components: {
    VueJsonPretty,
  },
  data () {
    return {
      expanded_items: [],
      static_agent_form:{
        'label':"",
        'role':"",
        'static_did':"",
        'static_key':"",
        'static_endpoint':"",
      },
    }
  },
  created: async function() {
    await this.ready();
    this.fetch_static_connections();
  },
  methods: {
    add: function(){
      let query_msg ={
        "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-static-connections/0.1/create-static-connection",
        "label": this.static_agent_form.label,
        "role": this.static_agent_form.role,
        "static_did": this.static_agent_form.static_did,
        "static_key": this.static_agent_form.static_key,
        "static_endpoint": this.static_agent_form.static_endpoint,
      }
      this.send_message(query_msg);
    },
    get_name: function(c) {
      return c.their_info.label; //i.connection.invitation_mode +" / "+ i.connection.their_role +" / "+ i.connection.created_at ;
    },
    copyItem: function(item,title){
      clipboard.writeText(item);
      this.$notify({
        type: 'success',
        title: 'Copied',
        message: 'This '+title+' has been copied to the clipboard.',
        duration: 2000
      });
    },
  }
}
</script>
