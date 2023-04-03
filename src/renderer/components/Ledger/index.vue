<template >
  <el-row>
    <p>Ledgers:</p>
    <el-collapse v-model="expanded_ledger_items">
      <div v-for="(ledger, key) in ledgers" :key="key">
        <el-collapse-item v-bind:title="'Name: ' + ledger.name" :name="key">
          <el-row>
            <div>
              <vue-json-pretty
                :deep=1
                :data="ledger">
              </vue-json-pretty>
              <el-form class="ledger-form">
                <el-form-item >
                  <el-button type="primary" @click="removeLedger(ledger)">delete</el-button>
                </el-form-item>
                <el-button v-on:click="collapse_expanded_ledger_item(key)">^</el-button>
              </el-form>
            </div>
          </el-row>
        </el-collapse-item>
      </div>
    </el-collapse>
    <p>Add new ledger:</p>
    <el-form :model=ledger_form>

        <span slot="label">Name:</span>
        <el-input v-model="ledger_form.name" style="width:100px;"> </el-input>
        <span slot="label">Url to genesis file:</span>
        <el-input v-model="ledger_form.gen_url" style="width:100px;"> </el-input>

      <el-form-item>
        <el-button type="primary" @click="addLedger()">add new ledger</el-button>
      </el-form-item>
    </el-form>
  </el-row>
</template>

<script>
import VueJsonPretty from 'vue-json-pretty';
import message_bus from '../../message_bus.js';

export const metadata = {
  menu: {
    label: 'Ledgers',
    icon: 'el-icon-connection',
    group: 'Agent to Agent',
    priority: 80,
    required_protocols: [
      {'https' : 'https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-ledgers/0.1'},
      {'did:sov' : 'did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-ledgers/0.1'}
    ]
  }
};

export default {
  name: 'ledgers',
  mixins: [
    message_bus()
  ],
  components: {
    VueJsonPretty,
  },
  data () {
    return {
      ledgers:{
        '12345234':{
          'id':'12345234',
          'name':'sov',
        },
        '22345234':{
          'id':'22345234',
          'name':'von',
        },
        '32345234':{
          'id':'32345234',
          'name':'bank',
        },
        '42345234':{
          'id':'42345234',
          'name':'adams',
        },
      },
      ledger_form:{
        'name':'',
        'gen_url':'',
      },
      expanded_ledger_items:[],
    }
  },
  created: function() {
    let component = this; // Safe rerefence to this
    this.$message_bus.$on(
      'https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-connections/0.1/invitation',
      msg => component.fetchAgentInvitations()
    );
    this.$message_bus.$on('invitations', () => component.fetchAgentInvitations());
  },
  methods: {
    async collapse_expanded_ledger_item(id){
      let index = this.expanded_ledger_items.indexOf(id)
      this.expanded_ledger_items.splice(index, 1);
    },
  },
  computed: {
    ledgerSchemas(ledger_name){
      console.log("ledger name:",ledger_name)
      let schemas = Object.keys(this.schemas).reduce((acc,key)=>
        ("ledgers" in this.schemas[key] && ledger_name in this.schemas[key].ledgers ?{
          ...acc,
          [key]: this.schemas[key]
        } : acc
        ), {})
      console.log("schemas",schemas)
      return schemas
    },
  }
}
</script>
