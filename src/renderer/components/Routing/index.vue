<template>
  <el-row>
    <div style="margin-bottom: 1em;">
      <nav class="navbar navbar-expand-lg navbar-light bg-light">
        <a class="navbar-brand" href="#">Mediators</a>
        <el-button
          type="primary"
          icon="el-icon-refresh"
          @click="fetch_mediators"></el-button>
      </nav>
      <el-collapse v-model="expanded_items">
        <ul class="list">
          <el-collapse-item
            v-for="m in mediators"
            v-bind:title="mediator_name(m)"
            :name="m.connection_id"
            :key="m.connection_id">
            <el-row>
              <div>
                <vue-json-pretty
                  :deep=1
                  :data="m">
                </vue-json-pretty>
              </div>
            </el-row>
          </el-collapse-item>
        </ul>
      </el-collapse>
      <nav class="navbar navbar-expand-lg navbar-light bg-light">
        <a class="navbar-brand" href="#">Client Routes</a>
        <el-button
          type="primary"
          icon="el-icon-refresh"
          @click="fetch_client_routes"></el-button>
      </nav>
      <el-collapse v-model="expanded_items">
        <ul class="list">
          <el-collapse-item
            v-for="r in routes"
            v-bind:title="route_name(r)"
            :name="r.recipient_key"
            :key="r.recipient_key">
            <el-row>
              <div>
                <vue-json-pretty
                  :deep=1
                  :data="r">
                </vue-json-pretty>
              </div>
            </el-row>
          </el-collapse-item>
        </ul>
      </el-collapse>
    </div>
    <el-form inline :model="request_mediation_form">
      <p>Request Mediation</p>
      <el-form-item label="Connection:" :label-width="formLabelWidth">
        <el-select
          v-model="request_mediation_form.connection_id"
          filterable
          no-data-text="No connections found"
          placeholder="Potential mediator...">
          <el-option
            v-for="connection in active_connections"
            :key="connection.connection_id"
            :label="connection.label"
            :value="connection.connection_id">
          </el-option>
        </el-select>
        <el-button
          type="primary"
          icon="el-icon-plus"
          @click="mediation_request_send">Request</el-button>
      </el-form-item>
    </el-form>
    <el-form :model="updaterouteform">
      <p>Add Route to connected mediator</p>
      <el-form-item label="Connection:" :label-width="formLabelWidth">
        <el-select
          v-model="updaterouteform.connection_id"
          filterable
          no-data-text="No connections found"
          placeholder="Connection">
          <el-option
            v-for="connection in active_connections"
            :key="connection.connection_id"
            :label="connection.label"
            :value="connection.connection_id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="DID:" :label-width="formLabelWidth">
        <el-select
          v-model="updaterouteform.did"
          value-key="did.did"
          filterable
          no-data-text="No DIDs found"
          placeholder="DID">
          <el-option
            v-for="did in dids"
            :key="did.did"
            :label="did_get_name(did)"
            :value="did">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="Action:" :label-width="formLabelWidth">
        <el-select
          v-model="updaterouteform.action">
          <el-option value="add"></el-option>
          <el-option value="remove"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="" :label-width="formLabelWidth">
        <el-button type="primary" @click="keylist_update_send">Go</el-button>
      </el-form-item>
    </el-form>
  </el-row>
</template>

<style></style>

<script>
import message_bus from '@/message_bus.js';
import share from '@/share.js';

const PROTOCOL = 'https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-routing/0.1';

export const metadata = {
  menu: {
    label: 'Routing',
    icon: 'el-icon-location-outline',
    group: 'Agent to Agent',
    priority: 55,
    required_protocols: [
    {'https' : 'https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-routing/0.1'},
    {'did:sov' : 'did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-routing/0.1'}
    ]
  }
};

export const shared = {
  data: {
    routes: [],
    mediators: []
  },
  listeners: {
    ['https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-routing/0.1/routes']: (share, msg) => {
      share.routes = msg.routes;
    },
    ['did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-routing/0.1/routes']: (share, msg) => {
      share.routes = msg.routes;
    },
    ['https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-routing/0.1/mediation-requests']: (share, msg) => {
      share.mediators = msg.requests;
    },
    ['did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-routing/0.1/mediation-requests']: (share, msg) => {
      share.mediators = msg.requests;
    }
  },
  methods: {
    fetch_client_routes: ({send}) => {
      send({"@type": `${PROTOCOL}/routes-get`});
    },
    fetch_mediators: ({send}) => {
      send({"@type": `${PROTOCOL}/mediation-requests-get`})
    }
  }
};

import VueJsonPretty from 'vue-json-pretty';

export default {
  name: 'routing',
  components: {
    VueJsonPretty,
  },
  mixins: [
    message_bus(),
    share({
      use: [
        'active_connections',
        'routes',
        'mediators',
        'dids',
      ],
      actions: ['fetch_connections', 'fetch_mediators', 'fetch_client_routes']
    })
  ],
  data: function() {
    return {
      expanded_items:[],
      updaterouteform: {
        connection_id: null,
        did: null,
        action: 'add',
      },
      request_mediation_form: {
        connection_id: null
      },
      formLabelWidth: '100px',
    }
  },
  created: async function() {
    await this.ready()
    this.load();
  },
  methods: {
    did_get_name: function(did) {
      if('metadata'in did && 'label' in did.metadata) {
        return 'did: ' + did.metadata.label;
      }
      return 'did: ' + did.did + ', vk: ' + did.verkey;
    },
    route_name: function(route) {
      let connection_label = 'Unknown';
      let matched_connection = this.active_connections.filter(
        c => c.connection_id == route.connection_id
      );
      if(matched_connection.length == 1){
        connection_label = matched_connection[0].label;
      }
      return `${connection_label}: ${route.recipient_key}`;
    },
    mediator_name: function(mediator) {
      let connection_label = 'Unknown';
      let matched_connection = this.active_connections.filter(
        c => c.connection_id == mediator.connection_id
      );
      if(matched_connection.length == 1){
        connection_label = matched_connection[0].label;
      }
      return connection_label;
    },
    keylist_update_send: function() {
      let msg = {
        "@type": `${PROTOCOL}/keylist-update-send`,
        "connection_id": this.updaterouteform.connection_id,
        "verkey": this.updaterouteform.did.verkey,
        "action": this.updaterouteform.action,
      };
      this.send_message(msg);
    },
    mediation_request_send: function() {
      let msg = {
        "@type": `${PROTOCOL}/mediation-request-send`,
        "connection_id": this.request_mediation_form.connection_id
      };
      this.send_message(msg);
    },
    load: function() {
      this.fetch_connections();
      this.fetch_client_routes();
    }
  },
}
</script>
