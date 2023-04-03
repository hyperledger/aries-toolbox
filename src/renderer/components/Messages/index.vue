<template>
  <el-row>
    <el-form @submit.native.prevent :inline="true">
      <el-form-item label="To:">
        <el-select
          v-model="connection_id"
          filterable
          no-data-text="No connections found"
          placeholder="Select Conversation"
          @change="connection_selected">
          <el-option
            v-for="connection in active_connections"
            :key="connection.connection_id"
            :label="connection.label"
            :value="connection.connection_id">
          </el-option>
        </el-select>
      </el-form-item>
    </el-form>
    <el-form @submit.native.prevent :inline="true" style="margin-bottom: 1em;">
      <el-input
        placeholder="Message"
        @keyup.enter.native="send"
        v-model="content"
        style="width:500px;">
      </el-input>
        <el-button
          type="primary"
          icon="el-icon-s-promotion"
          @click="send">Send</el-button>
    </el-form>
    <div id="message-container">
        <div
          v-for="m in messages"
          :key="m.message_id"
          :class="'basic_message basic_message-'+m.state">
          <div class="content">{{m.content}}</div>
          <div class="timestamp">{{nice_time(m.sent_time)}}</div>
        </div>
    </div>
  </el-row>
</template>

<style>
#message-container {
  display: flex;
  flex-direction: column;
}
.message-display {
  margin-bottom: 1em;
  border-bottom: 1px solid lightgrey;
  padding-bottom: 1em;
}
.basic_message:hover .timestamp {
  display: block;
}
.basic_message-recv {
  background-color: white;
  margin-right: auto;
  margin-bottom: 1em;
  padding: 1em;
  border: 1px solid lightgrey;
  border-radius: 4px;
  justify-content: flex-start;
}
.basic_message-sent {
  background-color: #ecf5ff;
  margin-left: auto;
  margin-bottom: 1em;
  padding: 1em;
  justify-content: flex-end;
  border: 1px solid #d9ecff;
  border-radius: 4px;
}
.basic_message-sent .timestamp {
  text-align: right;
}
.basic_message-recv .timestamp {
  text-align: left;
}

.timestamp {
  display: none;
  font-style: italic;
  font-size: small;
  color: grey;
}
</style>

<script>
import message_bus from '@/message_bus.js';
import share from '@/share.js';

export const protocol = 'https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-basicmessage/0.1';
export const metadata = {
  menu: {
    label: 'Messages',
    icon: 'el-icon-chat-line-square',
    group: 'Agent to Agent',
    priority: 45,
    required_protocols: [
      {'https' : 'https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-basicmessage/0.1'},
      {'did:sov' : 'did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-basicmessage/0.1'}
    ]
  },
  message_types: {
    new: protocol + '/new',
    send: protocol + '/send',
    sent: protocol + '/sent',
    get: protocol + '/get',
    messages: protocol + '/messages',
    delete: protocol + '/delete',
    deleted: protocol + '/deleted'
  }
};

export const shared = {
  data: {
    basic_messages: {},
    latest_message: null
  },
  listeners: {
    [metadata.message_types.sent]: (share, msg) => {
      if (!(msg.connection_id in share.basic_messages)) {
        share.$set(share.basic_messages, msg.connection_id, []);
      }
      share.basic_messages[msg.connection_id].push(msg.message);
    },
    [metadata.message_types.new]: (share, msg) => {
      if (!(msg.connection_id in share.basic_messages)) {
        share.$set(share.basic_messages, msg.connection_id, []);
      }
      share.basic_messages[msg.connection_id].push(msg.message);
      if (share.vm.$route.name !== 'a2a-message') {
        share.new_message_notify(msg.connection_id, msg.message);
      }
      share.latest_message = msg;
    },
    [metadata.message_types.messages]: (share, msg) => {
      share.$set(share.basic_messages, msg.connection_id, msg.messages);
    }
  },
  methods: {
    fetch_messages: function({send}, connection_id, page=0) {
      let limit = 30;
      let offset = 0;
      if (page !== 0) {
        offset = limit * page;
      }
      send({
        '@type': metadata.message_types.get,
        'connection_id': connection_id,
        'limit': limit,
        'offset': offset
      });
    },
    new_message_notify: function({share}, connection_id, msg) {
      let label = '[Unknown]';
      if (connection_id in share.id_to_connection) {
        label = share.id_to_connection[connection_id].label;
      }

      share.$notify({
        title: 'New message from ' + label,
        message: (text => {
          if (text.length > 30) {
            return text.slice(0, 30).trim() + '...';
          }
          return text;
        })(msg.content),
        duration: 4000
      });
    }
  }
};

export default {
  name: 'a2a-message',
  mixins: [
    message_bus(),
    share({
      use: ['basic_messages', 'active_connections', 'latest_message'],
      actions: ['fetch_connections', 'fetch_messages', 'new_message_notify']
    })
  ],
  data: function() {
    return {
      content: '',
      connection_id: '',
    }
  },
  watch: {
    latest_message: function(msg) {
      if (msg.connection_id != this.connection_id) {
        this.new_message_notify(msg.connection_id, msg.message);
      }
    }
  },
  computed: {
    messages: function() {
      if (!(this.connection_id in this.basic_messages)) {
        return [];
      }
      return this.basic_messages[this.connection_id].sort((lhs, rhs) => {
        return new Date(rhs.sent_time) - new Date(lhs.sent_time);
      });
    },
  },
  created: async function() {
    await this.ready()
    this.fetch_connections();
  },
  methods: {
    send: function() {
      if (!this.content) {
        this.content = '';
        return;
      }

      let msg = {
        "@type": metadata.message_types.send,
        "connection_id": this.connection_id,
        "content": this.content
      };
      this.send_message(msg);
      this.content = '';
    },
    connection_selected: function(selected) {
      this.fetch_messages(selected);
    },
    nice_time: function(timestamp) {
      return new Date(timestamp).toLocaleString(
        'en-US',
        {
          weekday: 'long',
          day: 'numeric',
          month: 'numeric',
          year: 'numeric',
          hour: 'numeric',
          minute: 'numeric'
        }
      );
    }
  }
}
</script>
