<template>
  <el-row>
    <div style="margin-bottom: 1em;">
      <nav class="navbar navbar-expand-lg navbar-light bg-light">
        <a class="navbar-brand" href="#">Mediation Clients</a>
        <el-button
          type="primary"
          icon="el-icon-refresh"
          @click="fetch_mediator_clients"></el-button>
      </nav>
      <el-collapse v-model="expanded_items">
        <ul class="list">
          <el-collapse-item
            v-for="client in mediator_clients"
            v-bind:title="client_name(client)"
            :name="client.mediation_id"
            :key="client.mediation_id">
            <el-row :key="client.mediation_id">
              <p>Status: {{client.state}}</p>
              <el-button v-if="client.state == 'request'" type="primary" @click="grant(client)">Grant</el-button>
              <el-button v-if="client.state == 'request'" type="danger" @click="deny(client)">Deny</el-button>
              <div>
                <vue-json-pretty
                  :deep=1
                  :data="client">
                </vue-json-pretty>
              </div>

            </el-row>
          </el-collapse-item>
        </ul>
      </el-collapse>
      <nav class="navbar navbar-expand-lg navbar-light bg-light">
        <a class="navbar-brand" href="#">Mediated Routes</a>
        <el-button
          type="primary"
          icon="el-icon-refresh"
          @click="fetch_mediated_routes"></el-button>
      </nav>
      <el-collapse v-model="expanded_items">
        <ul class="list">
          <el-collapse-item
            v-for="r in mediator_routes"
            v-bind:title="route_name(r)"
            :name="r.recipient_key"
            :key="r.connection_id">
            <el-row :key="r.connection_id">
              <div>
                <vue-json-pretty
                  :deep=2
                  :data="r">
                </vue-json-pretty>
              </div>
            </el-row>
          </el-collapse-item>
        </ul>
      </el-collapse>
    </div>
    <div v-if="get_connection().active_as_mediator == true">
      <el-alert
        title="This connection is set as the toolbox mediator."
        type="success"
        :closable="false">
      </el-alert>
    </div>
    <div style="margin-top: 5em; text-align: center;" v-else>
      <p>Mediation for the <strong>Toolbox</strong> can be requested directly from this agent, {{get_connection().label}}.</p>
      <el-button
        v-on:click="request_mediation"
        type="primary"
        >Request Mediation from {{get_connection().label}}</el-button>
    </div>
  </el-row>
</template>

<style>

</style>

<script>
import message_bus from '@/message_bus.js';
import share from '@/share.js';
import { mapActions } from "vuex"

const PROTOCOL = 'https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-mediator/0.1';
export const COORDINATE_MEDIATION = 'https://didcomm.org/coordinate-mediation/1.0';

export const metadata = {
  menu: {
    label: 'Mediation',
    icon: 'el-icon-guide',
    group: 'Agent to Agent',
    priority: 50,
    required_protocols: [
      {'https' : 'https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-mediator/0.1'},
      {'did:sov' : 'did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-mediator/0.1'}
    ]
  }
};

export const shared = {
  data: {
    mediator_routes: {},
    mediator_clients: [],
  },
  listeners: {
    ['https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-mediator/0.1/routes']: (share, msg) => {
      share.mediator_routes = msg.routes;
    },
    ['did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-mediator/0.1/routes']: (share, msg) => {
      share.mediator_routes = msg.routes;
    },
    ['https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-mediator/0.1//mediation-requests']: (share, msg) => {
      share.mediator_clients = msg.requests;
    },
    ['did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-mediator/0.1//mediation-requests']: (share, msg) => {
      share.mediator_clients = msg.requests;
    },
  },
  methods: {
    request_mediation: ({send}) => {
      send({
        "@type": `${COORDINATE_MEDIATION}/mediate-request`
      });
    },
    fetch_mediated_routes: ({send}) => {
      send({"@type": `${PROTOCOL}/routes-get`});
    },
    fetch_mediator_clients: ({send}) => {
      send({
        "@type": `${PROTOCOL}/mediation-requests-get`
      });
    }
  }
};

import VueJsonPretty from 'vue-json-pretty';

export default {
  name: 'mediator',
  components: {
    VueJsonPretty,
  },
  inject: ['get_connection'],
  mixins: [
    message_bus(),
    share({
      use: ['mediator_routes',  'mediator_clients', 'active_connections'],
      actions: [
        'request_mediation',
        'fetch_connections',
        'fetch_mediated_routes',
        'fetch_mediator_clients',
        'enable_as_mediator'
      ]
    })
  ],
  data: function() {
    return {
      expanded_items:[],
    }
  },
  created: async function() {
    await this.ready()
    this.load();
  },
  methods: {
    ...mapActions("Agents", ["update_agent"]),
    route_name: function(r){
      let connection_label = 'Unknown';
      let matched_connection = this.active_connections.filter(function(c){
        return c.connection_id == r.connection_id;
      });
      if(matched_connection.length == 1){
        connection_label = matched_connection[0].label;
      }
      return connection_label + ': ' + r.recipient_key;
    },
    client_name: function(client) {
      let connection_label = 'Unknown';
      let matched_connection = this.active_connections.filter(
        c => c.connection_id == client.connection_id
      );
      if(matched_connection.length == 1){
        connection_label = matched_connection[0].label;
      }
      return `${connection_label} (${client.state})`;
    },
    load: function() {
      this.fetch_connections();
      this.fetch_mediated_routes();
      this.fetch_mediator_clients();
    }
  }
}
</script>
