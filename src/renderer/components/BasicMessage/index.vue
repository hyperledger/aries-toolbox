<template>
  <el-row>
    <div style="margin-bottom: 1em;">
      <el-input
        placeholder="Message to send to the connected Agent"
        @keyup.enter.native="send"
        v-model="content"
        style="width:500px;">
        <el-button
          slot="append"
          type="primary"
          icon="el-icon-check"
          @click="send">Send</el-button>
      </el-input>
    </div>
    <div v-for="m in basicmessages.slice().reverse()" :key="m.msg['@id']">
      <div :class="'basicmessage-'+m.direction">{{m.msg.content}}</div>
    </div>
  </el-row>
</template>

<style>
.message-display {
  margin-bottom: 1em;
  border-bottom: 1px solid lightgrey;
  padding-bottom: 1em;
}
.basicmessage-Sent {
  background-color: white;
  margin-right: 4em;
  margin-bottom: 1em;
  padding: 1em;
  border: 1px solid lightgrey;
  border-radius: 4px;
}
.basicmessage-Received {
  background-color: lightblue;
  margin-left: 4em;
  margin-bottom: 1em;
  padding: 1em;
  text-align: right;
  border: 1px solid lightgrey;
  border-radius: 4px;
}
</style>

<script>
import message_bus from '@/message_bus.js';
import share from '@/share.js';

export const metadata = {
  menu: {
    label: 'Basic Message',
    icon: 'el-icon-chat-line-square',
    group: 'Toolbox to Agent',
    priority: 20,
    required_protocols: [
      {'did:sov' : 'did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/basicmessage/1.0'},
      {'https': 'https://github.com/hyperledger/aries-toolbox/tree/master/docs/basicmessage/1.0'}
    ]
  }
};

export const shared = {
  data: {
    basicmessages: [],
  },
  listeners: {
    'did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/basicmessage/1.0/message': (share, msg) => {
      share.basicmessages.push({
        'msg': msg,
        'direction': 'Received'
      });
    },
    'https://github.com/hyperledger/aries-toolbox/tree/master/docs/basicmessage/1.0/message': (share, msg) => {
      share.basicmessages.push({
        'msg': msg,
        'direction': 'Received'
      });
    },
    'send-message': (share, msg) => {
      if (msg['@type'] === 'did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/basicmessage/1.0/message' 
      || 'https://github.com/hyperledger/aries-toolbox/tree/master/docs/basicmessage/1.0/message') {
        share.basicmessages.push({
          'msg': msg,
          'direction': 'Sent'
        });
      }
    }
  }
};

export default {
  name: 'basic-message',
  mixins: [
    message_bus(),
    share({
      use: ['basicmessages']
    })
  ],
  data: function() {
    return {
      content: '',
    }
  },
  methods: {
    send: function() {
      let msg = {
        "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/basicmessage/1.0/message",
        "content": this.content
      };
      this.send_message(msg);
      this.content = '';
    }
  }
}
</script>
