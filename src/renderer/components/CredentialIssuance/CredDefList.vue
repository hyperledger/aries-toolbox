<template >
  <div>
    <nav class="navbar navbar-expand-lg navbar-light bg-light">
      <a class="navbar-brand" href="#">{{ title }}</a>
      <el-input
        v-if="retrievable"
        v-model="retrieve_cred_def_id"
        style="width: 300px;">
        <el-button
          slot="append"
          icon="el-icon-search"
          @click="get_cred_def">Retrieve</el-button>
      </el-input>
      <el-button
        v-if="can_create && list"
        type="primary"
        icon="el-icon-plus"
      @click="createFormActive = true">Create</el-button>
      <el-button
        type="primary"
        icon="el-icon-refresh"
        @click="$emit('cred-def-refresh',)"></el-button>
    </nav>
    <el-collapse v-model="expanded_items">
      <ul class="list">
        <el-collapse-item
          v-for="creddef in list"
          v-bind:title="creddef.cred_def_id"
          :name="creddef.cred_def_id"
          :key="creddef.cred_def">
          <el-row :key="creddef.cred_def">
            <ul>
              <li><strong>Schema ID:</strong> {{creddef.schema_id}}</li>
              <li><strong>Created:</strong> {{creddef.created_at}}</li>
              <li><strong>Attributes:</strong> <attributes :list="creddef.attributes" inline></attributes></li>
            </ul>
            <div>
              <vue-json-pretty
                :deep=0
                :deepCollapseChildren="true"
                :data="creddef">
              </vue-json-pretty>
            </div>
            <el-button v-on:click="collapse_expanded(creddef)">^</el-button>
          </el-row>
        </el-collapse-item>
      </ul>
    </el-collapse>
    <el-dialog title="Create New Credential Definition" :visible.sync="createFormActive" @close="deactivate()">
      <el-form :model="createForm">
        <el-form-item label="Schema:" :label-width="formLabelWidth">
          <el-select
            v-model="createForm.schema_id"
            filterable
            no-data-text="No schemas found"
            value-key="createForm.schema_id"
            placeholder="Select">
            <el-option
              v-for="schema in schemas"
              :key="schema.schema_id"
              :label="schema.schema_name +' '+ schema.schema_version"
              :value="schema.schema_id">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="Revocable?" :label-width="formLabelWidth">
          <el-checkbox id="support_revocation" v-model="createForm.support_revocation" />
        </el-form-item>
        <el-form-item label="Revocation Registry Size" :label-width="formLabelWidth">
          <el-input-number id="revocation_registry_size" v-model="createForm.revocation_registry_size" :disabled="!createForm.support_revocation" min=4 max=32769 />
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="deactivate()">Cancel</el-button>
        <el-button type="primary" @click="publish_cred_def">Confirm</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import VueJsonPretty from 'vue-json-pretty';
import message_bus from '../../message_bus.js';
import share from '../../share.js';
import Attributes from './Attributes.vue';

export default {
  name: 'cred-def-list',
  props: [
    'retrievable',
    'title',
    'list',
    'can_create',
  ],
  mixins: [
    message_bus({
      events: {
        "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-credential-definitions/0.1/credential-definition-id":
        (v, msg) => setTimeout(v.fetch_cred_defs, 4500),
        "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-credential-definitions/0.1/credential-definition-id":
        (v, msg) => setTimeout(v.fetch_cred_defs, 4500),
        "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-credential-definitions/0.1/credential-definition-id":
        (v, msg) => setTimeout(v.fetch_cred_defs, 4500),
        "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-credential-definitions/0.1/credential-definition":
        (v, msg) => setTimeout(v.fetch_cred_defs, 4500),
      }
    }),
    share({
      use: [
        'schemas',
      ],
      actions: ['fetch_cred_defs'],
    })
  ],
  components: {
    VueJsonPretty,
    Attributes,
  },
  data () {
    return {
      expanded_items:[],
      createFormActive: false,
      createForm: {
        schema_id: '',
        support_revocation: false,
        revocation_registry_size: 100
      },
      retrieve_cred_def_id: '',
      formLabelWidth: '95px'
    }
  },
  created: async function() {
    await this.ready()
    this.fetch_cred_defs();
  },
  methods: {
    publish_cred_def: function(form) {
      let query_msg = {
        "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-credential-definitions/0.1/send-credential-definition",
        "schema_id": this.createForm.schema_id,
        "support_revocation": this.createForm.support_revocation,
      };
      if (this.createForm.support_revocation) {
        query_msg["revocation_registry_size"] = this.createForm.revocation_registry_size;
      }
      this.send_message(query_msg);
      this.createFormActive = false;
      this.createForm.schema_id = '';
      this.createForm.support_revocation = false;
      this.createForm.revocation_registry_size = 100;
    },
    get_cred_def: function() {
      let query_msg = {
        "@type": "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-credential-definitions/0.1/credential-definition-get",
        "cred_def_id": this.retrieve_cred_def_id,
      };
      this.send_message(query_msg);
      this.retrieve_cred_def_id = '';
    },
    collapse_expanded: function(creddef){
      this.expanded_items = this.expanded_items.filter(
        item => item != creddef.cred_def_id
      );
    },
    deactivate: function() {
      this.createFormActive = false;
      this.createForm.schema_id = '';
      this.createForm.support_revocation = false;
      this.createForm.revocation_registry_size = 100;
    },
  }
}
</script>
