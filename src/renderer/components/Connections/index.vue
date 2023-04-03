<template>
  <el-row :key="connection">
    <connection-list
      title="Active Connections:"
      editable="true"
      :list="active_connections"
      @connection-editted="update_connection"
      @connection-deleted="delete_connection"
      @refresh="fetch_connections"></connection-list>
    <connection-list
      title="Pending Connections:"
      editable="true"
      :list="pending_connections"
      @connection-editted="update_connection"
      @connection-deleted="delete_connection"></connection-list>
    <connection-list
      title="Failed Connections:"
      editable="false"
      :list="failed_connections"
      @connection-editted="update_connection"
      @connection-deleted="delete_connection"></connection-list>

    <p>Add connection from invitation:</p>
    <el-form @submit.native.prevent>
      <el-form-item
        label="Invitation URL:">
        <el-input
          style="width: 300px;"
          v-model="invitation">
          <el-button
            slot="append"
            type="primary"
            icon="el-icon-plus"
            @click="recieve_invitation">Add</el-button>
        </el-input>
      </el-form-item>
    </el-form>
  </el-row>
</template>

<script>
import ConnectionList from './ConnectionList.vue';
import share from '@/share.js';
import message_bus from '@/message_bus.js';

export const metadata = {
  menu: {
    label: 'Connections',
    icon: 'el-icon-user',
    group: 'Agent to Agent',
    priority: 30,
    required_protocols: [
      {'did:sov' : 'did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-connections/0.1'},
      {'https': 'https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-connections/0.1'}
    ]
  }
};

export const shared = {
  data: {
    connections_old: []
  },
  computed: {
    active_connections_old: function() {
        return Object.values(this.connections_old).filter(
          conn => {
            if (!("state" in conn)) {
              return false;
            }
            return conn.state === "active" || conn.state === "response"
          }
        );
    },
    id_to_connection: function(connection_id) {
      let map = {};
      this.connections_old.forEach((connection) => {
        map[connection.connection_id] = connection;
      })
      console.log(map);
      return map;
    }
  },
  listeners: {
    "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-connections/0.1/connection-list":
    (share, msg) => share.connections_old = msg.results,
    "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-connections/0.1/connection-list":
    (share, msg) => share.connections_old = msg.results,
    "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-connections/0.1/connection":
    (share, msg) => share.fetch_connections(),
    "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-connections/0.1/connection":
    (share, msg) => share.fetch_connections(),
    "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-connections/0.1/ack":
    (share, msg) => share.fetch_connections(),
    "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-connections/0.1/ack":
    (share, msg) => share.fetch_connections(),
  },
  methods: {
    fetch_connections: ({send}) => {
      send({
        "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-connections/0.1/connection-get-list",
      });
    }
  }
};

export default {
  name: 'connections',
  components: {
    ConnectionList
  },
  mixins: [
    message_bus(),
    share({
      use: ['connections_old', 'active_connections_old'],
      actions: ['fetch_connections']
    })
  ],
  data: function() {
    return {
      'invitation': '',
    }
  },
  created: async function() {
    await this.ready();
    this.fetch_connections();
  },
  computed: {
    pending_connections: function() {
      return Object.values(this.connections_old).filter(
        conn => "state" in conn &&
        conn.state !== "active" &&
        conn.state !== "invitation" &&
        conn.state !== "error"
      );
    },
    failed_connections: function() {
      return Object.values(this.connections_old).filter(
        conn => "state" in conn && conn.state === "error"
      );
    }
  },
  methods: {
    update_connection: function(form) {
      let query_msg = {
        "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-connections/0.1/update",
        "connection_id": form.connection_id,
        "label": form.label,
        "role": form.role,
      };
      this.send_message(query_msg);
    },
    delete_connection: function(connection) {
      let query_msg = {
        "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-connections/0.1/delete",
        "connection_id": connection.connection_id,
      };
      this.send_message(query_msg);
    },
    recieve_invitation: function() {
      let receive_invite_msg = {
        "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-connections/0.1/receive-invitation",
        "invitation": this.invitation,
        "accept": "auto"
      };
      this.send_message(receive_invite_msg);
      this.invitation = "";
      setTimeout(() => {
        return this.fetch_connections();
      }, 4000);
    },
  },
}
</script>
