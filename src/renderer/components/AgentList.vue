<template>
  <el-row>
    <nav class="navbar navbar-expand-lg navbar-light bg-light">
      <a class="navbar-brand" href="#">Aries Toolbox</a>
    </nav>

    <el-card shadow="never" class="agent-card" v-for="a in agent_list" v-bind:key="a.id">
      <span slot="header"><strong>{{a.label}}</strong></span>
      <div>
        <div v-show="a.active_as_mediator" class="mediator">Mediator</div>
        <el-button type="text" @click="openConnection(a)">Open</el-button>
        <el-button type="text" @click="deleteConnection(a)">Delete</el-button>
      </div>
    </el-card>

    <el-card shadow="never" class="function_card" id="new_agent_connection">
      <span slot="header">New Agent Connection</span>
      <div>
        <el-form :inline="true">
          <el-input v-model="new_agent_invitation" placeholder="Paste agent invitation"></el-input>
          <el-button type="primary" @click="connect_clicked">Connect</el-button>
        </el-form>
      </div>
      <el-alert v-show="invitation_error != ''"
        title="Invitation Error"
        type="error"
        :description="invitation_error"
        show-icon>
      </el-alert>
    </el-card>
    <el-card shadow="never" class="function_card" id="new_agent_invitation" v-show="hasMediator && false">
      <span slot="header">New Agent Invitation</span>
      <div>
        <el-form :inline="true">
          <el-input v-model="new_invitation_label" placeholder="Label"></el-input>
          <el-button type="primary" @click="generate_invitation">Generate</el-button>
        </el-form>
      </div>
      <el-alert v-show="invitation_error != ''"
        title="Invitation Error"
        type="error"
        :description="invitation_error"
        show-icon>
      </el-alert>
    </el-card>
    <el-card shadow="never" class="function_card" id="new_mediator_connection">
      <span slot="header">Connect to Mediator</span>
      <div>
        <el-form :inline="true">
          <el-input v-model="new_mediator_invitation" placeholder="Paste mediator invitation"></el-input>
          <el-button type="primary" @click="process_mediator_invitation">Connect</el-button>
        </el-form>
      </div>
      <el-alert v-show="mediation_error != ''"
        title="Invitation Error"
        type="error"
        :description="mediation_error"
        show-icon>
      </el-alert>
    </el-card>
  </el-row>
</template>

<script>
const electron = require('electron');
const bs58 = require('bs58');
const rp = require('request-promise');
const DIDComm = require('encryption-envelope-js');
//import DIDComm from 'didcomm-js';
import { mapState, mapActions, mapGetters } from "vuex"
import { new_connection } from '../connection_detail.js';
import message_bus from '@/message_bus.js';
import { base64_decode, base64_encode } from '../base64.js';
import { from_store } from '../connection_detail.js';
const uuidv4 = require('uuid/v4');
const coordinate_mediation =
  (type) => `did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/coordinate-mediation/1.0/${type}`;

export default {
  name: 'agent-list',
  components: {  },
  mixins: [
    message_bus({ events: {
      'toolbox-mediator-change': (vm) => {
        // broadcast via ipc to all the other windows
        for(const [key, win] of Object.entries(vm.agent_windows)){
          win.webContents.send("toolbox-mediator-change", {});
        }
      }
    }})
  ],
  computed: {
    ...mapState("Agents", ["agent_list"]),
    ...mapGetters("Agents", ["get_agent"]),
    hasMediator: function(){
      return this.agent_list.find(a => a.active_as_mediator === true);
    }
  },
  watch: {
    agent_list(newValue, oldValue) {
      // Do whatever makes sense now
      let mediator_agent = newValue.find(a => a.active_as_mediator === true);
      if (mediator_agent) {
        this.mediatorConnect(mediator_agent);
      } else {
        //mediator was deleted
        this.mediatorCleanup();
      }
      this.$message_bus.$emit('toolbox-mediator-change');
    },
  },
  methods: {
    ...mapActions("Agents", ["add_agent", "update_agent", "delete_agent"]),

    openConnection: async function(a) {
      const modalPath = process.env.NODE_ENV === 'development'
        ? 'http://localhost:9080/#/agent/'+a.id
        : `file://${__dirname}/index.html#agent/`+a.id;
      let win = new electron.remote.BrowserWindow({
        width: 1000,
        height: 600,
        webPreferences: {
          webSecurity: false,
          nodeIntegration: true
        }
      })

      // TODO: Get the close handler to work to keep track of open windows only
      win.on('close', function () {
        win = null;
        delete this.agent_windows[a.id];
      });
      win.loadURL(modalPath);
      let window_key = a.my_key_b58.publicKey;
      this.agent_windows[window_key] = win;

    },

    deleteConnection: async function(a){
      this.delete_agent(a);
    },

    mediatorConnect: async function(mediator_agent){
      let vm = this; //hold reference
      let agent_info = await this.get_agent(mediator_agent.id);
      this.mediator_connection = from_store(agent_info);
      this.mediator_connection.enable_return_route();
      this.mediator_connection.unpacked_processor = this.mediatorInbound(this.mediator_connection);
      //start poll timer
      if(this.mediator_connection.needs_return_route_poll()){
        this.return_route_poll_timer = setInterval(this.return_route_poll, 10000);
      }
      //TODO: open connection?
      this.mediator_connection.on_disconnect = function(){
        vm.mediator_connection.send_message({
          "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/trust_ping/1.0/ping",
          "response_requested": false
        });
      };
      this.mediator_connection.send_message({
        "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/trust_ping/1.0/ping",
        "response_requested": false
      });
      console.log("mediator connected", this.mediator_connection);
    },

    /**
     * Process messages received through mediator.
     */
    mediatorInbound: function(connection) {
      return async (packed_msg) => {
        console.log("mediator inbound message", packed_msg);
        let recipients = await connection.extract_packed_message_recipients(packed_msg);
        // Allow the AgentList to obtain a message from the mediator through the
        // message_bus
        let found = recipients.find(
          r => r === connection.my_key.publicKey_b58
        );
        if(found != null) {
          console.log('Unpacking and notifying on the message bus');
          let msg = await connection.unpackMessage(packed_msg);
          console.log(this.$message_bus);
          this.$message_bus.$emit(msg['@type'], msg);
          console.log('Got message from mediator for toolbox:', msg);
        }
        console.log("inbound message recipients", recipients);
        // TODO: get inbound message to proper agent window. this needs to happen even if window not open.
        // store message
        let message_uuid = uuidv4();

        // send message
        let window_key = recipients[0];
        // TODO: detect lack of open window.
        let window = this.agent_windows[window_key];
        let msg_bundle = {
            id: message_uuid,
            recipient: recipients[0],
            msg: packed_msg
          };
        if(window !== undefined){
          window.webContents.send("inbound_message", msg_bundle);
          console.log("delivered", msg_bundle);
        } else {
          // Queue message for when window opens
          if(!(window_key in this.window_message_queue)){
            this.window_message_queue[window_key] = [];
          }
          this.window_message_queue[window_key].push(msg_bundle);
          console.log("queued", msg_bundle);
        }
        //TODO: deliver queued message after window opens.
      };
    },

    /**
     * Tear down mediator info after mediator connection deletion.
     */
    mediatorCleanup: function(){
      if(
        this.mediator_connection &&
        this.mediator_connection.needs_return_route_poll &&
        this.mediator_connection.needs_return_route_poll()
      ) {
        clearInterval(this.return_route_poll_timer)
      }
      this.mediator_connection = null;
    },

    /**
     * Generate an invitation for connecting to the toolbox through mediation.
     */
    generate_invitation: function(){
      // TODO Implement
    },

    /**
     * Inform the mediator of a new key to route to this agent.
     * @param {string} verkey Key to add to mediator routes.
     */
    async add_route_to_mediator(verkey) {
      // prepare message
      let msg = {
        "@type": coordinate_mediation('keylist-update'),
        updates: [
          {
            recipient_key: verkey,
            action: "add"
          }
        ]
      };
      this.mediator_connection.send_message(msg);
      let response = await this.message_of_type(
        coordinate_mediation('keylist-update-response')
      );
      for (let update of response.updated) {
        console.log(`${update.action}(${update.recipient_key}): ${update.result}`);
        // TODO do something with errors
      }
    },

    async send_mediation_request(connection) {
      // prepare message
      let msg = {
        "@type": coordinate_mediation('mediate-request'),
      };
      await connection.send_message(msg);
      try {
        let response = await this.message_of_type(
          coordinate_mediation('mediate-grant'), 1000
        );
        connection.active_as_mediator = true;
        connection.mediator_info = {
          endpoint: response.endpoint,
          routing_keys: response.routing_keys
        };
        this.update_agent(connection.to_store());
        console.log("connection to mediate through", connection, connection.mediator_info);
      } catch (err) {
        console.error("Encountered error while attempting to establish mediation:", err)
      }
    },

    async process_mediator_invitation() {
      let connection = await this.new_agent_invitation_process(this.new_mediator_invitation);
      connection.unpacked_processor = this.mediatorInbound(connection);
      await this.send_mediation_request(connection);
      this.mediatorConnect(connection);
      this.new_mediator_invitation = "";
    },

    async connect_clicked() {
      this.invitation_error = "";
      try {
        await this.new_agent_invitation_process(this.new_agent_invitation);
      } catch (err) {
        console.log("request post err", err);
        this.invitation_error = err.message;
      }
      this.new_agent_invitation = "";
    },

    async new_agent_invitation_process(invitation){
      //process invite, prepare request
      //extract c_i param
      function getUrlVars(url) {
        var vars = {};
        var parts = url.replace(/[?&]+([^=&]+)=([^&]*)/gi, function(m,key,value) {
          vars[key] = value;
        });
        return vars;
      }

      var invite_b64 = getUrlVars(invitation)["c_i"];
      //base 64 decode
      var invite_string = base64_decode(invite_b64);
      var invite = JSON.parse(invite_string);

      //make a did
      const didcomm = new DIDComm.DIDComm();
      await didcomm.Ready;
      const toolbox_did = await didcomm.generateKeyPair();
      toolbox_did.did = bs58.encode(Buffer.from(toolbox_did.publicKey.subarray(0, 16)));
      toolbox_did.publicKey_b58 = bs58.encode(Buffer.from(toolbox_did.publicKey));
      toolbox_did.privateKey_b58 = bs58.encode(Buffer.from(toolbox_did.privateKey));

      let service_endpoint_block = {
        "id": toolbox_did.did + ";indy",
        "type": "IndyAgent",
        "recipientKeys": [toolbox_did.publicKey_b58],
        "serviceEndpoint": ""
      };

      if(this.mediator_connection && this.mediator_connection.active_as_mediator){
        let mediator_agent = this.agent_list.find(a => a.active_as_mediator === true);
        if(mediator_agent != null) {
          service_endpoint_block["routingKeys"] = mediator_agent.mediator_info.routing_keys || [];
          service_endpoint_block["serviceEndpoint"] = mediator_agent.mediator_info.endpoint;
        }
        console.log('Informing mediator about new key to route...');
        await this.add_route_to_mediator(toolbox_did.publicKey_b58);
      }

      var req = {
        "@id":  (uuidv4().toString()),
        "~transport": {
          "return_route": "all"
        },
        "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/connections/1.0/request",
        "label": "ToolBox",
        "connection": {
          "DID": toolbox_did.did,
          "DIDDoc": {
            "@context": "https://w3id.org/did/v1",
            "id": toolbox_did.did,
            "publicKey": [{
              "id": toolbox_did.did + "#keys-1",
              "type": "Ed25519VerificationKey2018",
              "controller": toolbox_did.did,
              "publicKeyBase58": toolbox_did.publicKey_b58
            }],
            "service": [service_endpoint_block]
          }
        }
      };
      console.log("Exchange Request", req);

      //send request, look for response
      const packedMsg = await didcomm.packMessage(JSON.stringify(req), [bs58.decode(invite.recipientKeys[0])], toolbox_did, true);
      console.log("Packed Exchange Request", packedMsg);

      // this code assumes that the response comes via return-route on the post.
      const res = await rp({
        method: 'POST',
        uri: invite.serviceEndpoint,
        body: packedMsg,
      });
      const unpackedResponse = await didcomm.unpackMessage(res, toolbox_did);
      const response = JSON.parse(unpackedResponse.message);
      //TODO: Validate signature against invite.
      let buff = new Buffer(response['connection~sig'].sig_data, 'base64');
      let text = buff.toString('ascii');
      //first 8 chars are a timestamp for the signature, so we ignore those before parsing value
      response.connection = JSON.parse(text.substring(8));
      console.log("response message", response);
      let connection_detail = new_connection(invite.label, response.connection.DIDDoc, toolbox_did);
      console.log("connection detail", connection_detail);
      this.add_agent(connection_detail.to_store());
      return connection_detail;
    },
  },
  created: async function(){
    let mediator_agent = this.agent_list.find(a => a.active_as_mediator === true);
    if(mediator_agent == null) {
      return; //no connection needed
    }
    this.mediatorConnect(mediator_agent);
  },
  beforeDestory: function(){
    this.mediatorCleanup();
  },
  data() {
    return {
      new_agent_invitation: "",
      new_mediator_invitation: "",
      invitation_error: "",
      mediation_error: "",
      new_invitation_label: "",
      agent_windows: {},
      mediator_connection: {},
      window_message_queue: {},
    }

  }
}
</script>

<style>
.agent-card:first-of-type {
  margin-top: .5em;
}
.agent-card, .function_card {
  margin: .5em 1em;
  border: 1px solid rgba(0, 0, 0, 0.125);
}
.agent-card .el-card__header,
.function_card .el-card__header {
  padding: .75rem 1.25rem;
  background-color: rgba(0, 0, 0, 0.03);
  border-bottom: 1px solid rgba(0, 0, 0, 0.125);
}
.agent-card .el-card__body {
  padding: 0 1em;
}
  .mediator {
    font-size: .9em;
    font-style: italic;
    text-align: right;
    float:right;
    margin-top: 10px;
  }
</style>
