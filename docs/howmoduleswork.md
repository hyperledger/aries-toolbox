# How Modules Work

Each UI module in the toolbox is represented by an entry in the left menu. The menu is created at runtime from a list of the components present and the connected agent's supported protocols. This guide is intended to document how to create new modules.

In addition to the information provided here, look at a few of the existing modules to understand how a complete module works. New components are most easily created by copying an existing module and making the necessary modifications.

### File Location

Each module exists as a folder inside `src/renderer/components/`. The main component for the module must be named `index.vue`. Additional Vue components can be named as appropriate, and stored in the same folder.

### Index Component

Each `index.vue` component file works as a standard vue component file, with a few additions detailed below.

#### metadata Export

An additional `metadata` export informs the toolbox about the Module name and requirements.

```javascript
export const metadata = {
    menu: {
      label: 'Module Name', 
      icon: 'el-icon-user',
      group: 'Agent to Agent',
      priority: 30,
      required_protocols: [
        {'https' : 'https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-connections/0.1'},
        {'did:sov' : 'did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-connections/0.1'}
      ]
    }
  };

```
**label** will appear in the menu.

**icon** must be an icon name from the [Element UI Icon Library](https://element.eleme.io/#/en-US/component/icon). This will also be used in the left menu.

**group** organizes the menu. Current groups are `Agent to Agent` and `Toolbox to Agent`.

**priority** sorts the menu item within the group. Lower numbers are higher in the list. Try to space your number to allow others to place there module as desired.

**required_protocols** is the list of protocol family prefixes required for the module to function. If all the protocols listed are not supported by the agent (via the discover_features protocol), the menu option will not be shown.

#### shared Export

The additional `shared` export contains data, handlers, and logic to handle inbound messages for this module. The following example is from the `Connections` module:

```javascript
export const shared = {
  data: {
    connections: []
  },
  computed: {
    active_connections: function() {
        return Object.values(this.connections).filter(
            conn => "state" in conn && conn.state === "active"
        );
    },
  },
  listeners: {
    "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-connections/0.1/connection-list":
    (share, msg) => share.connections = msg.results,
    "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-connections/0.1/connection-list":
    (share, msg) => share.connections = msg.results,
    "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-connections/0.1/connection":
    (share, msg) => share.fetch_connections(),
    "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/admin-connections/0.1/connection":
    (share, msg) => share.fetch_connections(),
    "https://github.com/hyperledger/aries-toolbox/tree/master/docs/admin-connections/0.1/ack":
    (share, msg) => share.fetch_connections(),
    "did:sov:BzCbsNYhMrjHiqZDTUASHg;spec/docs/admin-connections/0.1/ack":
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
```

**data** contains session state for the module. Declaring this here allows the message handlers to read or modify the state. Use of these elements is enabled through a mixin to the main component, detailed below.

**computed** declares any computed values.

**listeners** contains inbound message handlers. These functions can modify values declared in **data**, but through the `share` handler attribute as shown.

**methods** can be called by inbound message handlers as well as from the main component. The `send` method is used to send an outbound message to the connected agent.

#### mixins

The main vue component uses a few mixins to tie things together. The example below is also from the `Connections` module:

```javascript
  mixins: [
    message_bus(),
    share({
      use: ['connections', 'active_connections'],
      actions: ['fetch_connections']
    })
  ],
```

`message_bus()` takes no arguments, but provides the `send_message()` function used to send outbound messages.

`share()` takes an object with the following elements:

`use` is a list of the data and computed elements from the shared Export. This imports the values listed for use with `this.` in the main component.

`actions` imports actions declared in the shared Export. This allows them to be called from within the main component.