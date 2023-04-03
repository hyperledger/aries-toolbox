<template>
  <el-row>
    <nav class="navbar navbar-expand-lg navbar-light bg-light">
      <a class="navbar-brand" href="#">Discover Feature</a>
      <el-button
        type="primary"
        icon="el-icon-refresh"
        @click="fetch_protocols"></el-button>
    </nav>
    <table class="table table-sm">
      <tr>
        <th>Protocol</th>
        <th>Roles</th>
      </tr>
      <tr v-for="p in protocols" :key="p.pid">
        <td>{{p.pid}}</td>
        <td>{{p.roles}}</td>
      </tr>
    </table>
  </el-row>
</template>

<script>
import share from '@/share.js';

export const metadata = {
  menu: {
    label: 'Discover Features',
    icon: 'el-icon-discover',
    group: 'Toolbox to Agent',
    priority: 5,
    required_protocols: [
    ]
  }
};

export const shared = {
  data: {
    protocols: []
  },
  computed: {

  },
  listeners: {
    "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/discover-features/1.0/disclose":
    (share, msg) => share.protocols = msg.protocols,
    "https://github.com/hyperledger/aries-toolbox/tree/master/docs/discover-features/1.0/disclose":
    (share, msg) => share.protocols = msg.protocols,
  },
  methods: {
    fetch_protocols: ({send}) => {
      send({
        "@type": "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/discover-features/1.0/query",
        "query": "*"
      });
    }
  }
};

export default {
  name: 'feature-discovery',
  mixins: [
    share({
      use: ['protocols'],
      actions: ['fetch_protocols'],
    })
  ],
  created: async function() {
    await this.ready();
    this.fetch_protocols();
  },
  methods: {

  }
}
</script>
