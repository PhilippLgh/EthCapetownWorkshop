# EthCapetownWorkshop

## Installation

Grid consists of 2 parts: the main application and the UI.
In order to hack on Grid we want to start it in `dev mode`, which is why we clone both repos and run from source:

```
mkdir grid
cd grid
```
Install and run Grid UI:
```
git clone https://github.com/ethereum/grid-ui.git
cd grid-ui
yarn && yarn start
```
Install and run Grid:
```
cd ..
git clone https://github.com/ethereum/grid.git
yarn && yarn start:dev
```

There are also prebuilt and pre-bundled installers and binaries available to have a convenient way to run the `production` version. The binaries can be found on the [Grid Website](https://grid.ethereum.org/) or the [GitHub releases](https://github.com/ethereum/grid/releases).

## What is Grid?

“Often referred to […] as “digital frontier", the Grid was made to provide an experimental platform where all forms of research could be carried out at unparalleled speeds.” - [Tron Movie](https://tron.fandom.com/wiki/Grid)

One way to describe Ethereum Grid is a platform to run experiments, create prototypes (and hackathon projects) or develop fully working apps for the many available clients in the ecosystem.
Another description could be that Ethereum Grid is the control center for all kinds of clients and Ethereum core binaries.

![Grid Screenshot](/assets/Grid-Screenshot.png)

With Grid, clients can be downloaded, configured, and started all in one place. But even more than this, Grid serves as an Ethereum provider, which means that once a client is configured and started, DApps can connect to Grid and share the connection to the Ethereum network.

It is ideal for people who want to run a full node, have a convenient and secure way to update their binaries, and don't want to rely on centralized 3rd party services like Infura / Metamask.

Grid can host and launch other interfaces and the functionality can be extended and adjusted through apps and plugins.

![Grid Tray Integration](/assets/Grid-Tray.png)

## How is Grid related to Web3, Mist or Metamask?

The Ethereum Mist browser can be considered a predecessor of Grid. However, [Mist is no longer developed](https://medium.com/@avsa/sunsetting-mist-da21c8e943d2) and the lessons learned are the main inspiration and *raison d'être* for Grid.

Web3 browsers, such as Mist and Metamask, share the same core mechanics: an Ethereum client is started, a connection is established to the Ethereum network and the client process, and an API (Web3) is provided to some App or DApp for easy interaction with the network, smart contracts, et cetera.

Grid has the same core mechanics. However, instead of connecting to a centralized 3rd-party-hosted client in the cloud like Infura, it gives the user full control over the client, version and configuration. Everything is run locally. And Grid tries to make this as easy as possible.

Grid focuses on a more experienced audience and the interactions with the clients happen on lower levels (no Web3) and are more powerful but also less convenient. Clients are not limited to any specific type and Grid's functionality can be extended.

If your goal is to write an application like CryptoKitties, you should probably consider Metamask, Opera, Status or any other Web3 enabled browser for development and testing.

However, if you plan to develop a visualization for network behavior & events, a P2P sharing app for Swarm, a signer UI for Clef, your own wallet, dev tools or test suites to compare different spec implementations then you will probably want to use Grid to accelerate the development.

If you don't want to host your app on a server, but have it packaged and deployed in a more decentralized fashion, then Grid, `ethpkg` and `electron-app-manager` can help you with that as well.

## Extending Grid

There are currently two ways to extend Grid's functionality or integrate with it:

1. Client plugins
2. Web Apps

Client plugins specify repositories of Ethereum clients and metadata to download and run them. Grid can discover and load these plugin and will try to generate UI elements such as config forms for these clients.

Web apps are UI frontends written with Web technologies which can be hosted inside of Grid UI. They get access to the loaded clients, can interact through IPC with client processes, and listen to state changes through events.

### Plugins

Grid uses a plugin architecture to integrate, query releases and interact with various Ethereum clients. To define a new client all you need to do is to add a new file to the [`ethereum_clients/client_plugins/`](https://github.com/ethereum/grid/tree/master/ethereum_clients/client_plugins) directory within Grid.
Grid will automatically detect the client and provide UI elements to manage releases for the specified client.

#### Example 1: Clef

Here is an example for a Grid plugin. The below plugin structure specifies the [Ethereum Clef](https://github.com/ethereum/go-ethereum/tree/master/cmd/clef) integration and was taken from: [https://github.com/ethereum/grid/blob/master/ethereum_clients/client_plugins/clef.js](https://github.com/ethereum/grid/blob/master/ethereum_clients/client_plugins/clef.js)

```
const platform = process.platform === 'win32' ? 'windows' : process.platform

module.exports = {
  type: 'signer',
  order: 4,
  displayName: 'Clef',
  name: 'clef',
  repository: 'https://gethstore.blob.core.windows.net',
  filter: {
    version: '>=1.9.0' // only included in alltool package after (>=) 1.9.0
  },
  prefix: `geth-alltools-${platform}`,
  binaryName: process.platform === 'win32' ? 'clef.exe' : 'clef'
}
```

#### Example 2:  Swarm

```
module.exports = {
  type: 'storage',
  order: 5,
  displayName: 'Swarm',
  name: 'swarm',
  // https://swarm-gateways.net/bzz:/theswarm.eth/downloads/
  repository: 'https://gethstore.blob.core.windows.net'
}
```

#### Plugin Properties

The `type` field is used to specify one of the possible types [`client`, `signer`, `storage`]. 

`order` is an optional field to specify a position where the client should be displayed within the UI.

The `displayName` is the name that is shown to the user while the `name` property is used internally to query and find certain plugins by name.

The `repository` field is required to discover and download different releases.

`filter` options can be used to exclude or require certain keywords within the file name or version numbers and ranges that restrict the displayed download options.
`Filters` are applied after a release feed was downloaded which can be a heavy task. However, the filtering process can be sped up with `prefix` which uses server side API's to not even consider certain GitHub release assets or Azure hosted binaries as possible releases.

Grid (electron-app-manager) works on packages which are compressed container or archive files like `.zip` or `.tar`. The `binaryName` option specifies the name of the executable binary in the package. This is important because archives can contain more than one file or have nested subdirectories, and binaries have platform specific extensions (.exe).

`resolveIpc` is used to defined a method that returns a valid IPC endpoint, so that Grid can establish a secure connection to the spawned client process.

`flags` is an object that specifies the various command line arguments a client accepts. This is used to generate the configuration view:

Example:
```
flags: {
  '--datadir': {
    type: 'path',
    name: 'Data Directory',
    configKey: 'dataDir'
  },
  '--syncmode': {
    type: 'select',
    name: 'Sync Mode',
    configKey: 'syncMode',
    options: ['fast', 'light', 'full']
  },
  ...
 
```

The generated config fields in Grid UI:

![Grid Geth Config](/assets/Grid-Geth-Config.png)


>Grid uses `electron-app-manager` under the hood which handles all the download and version management of the binaries, so if some values are missing it can be helpful to check the docs of it as well.

#### Client Scripts

Some clients don't have executable binaries and require a runtime such as Python or Node.js (`nodex client.js`). These clients are **currently not supported**. A way around this restriction is to bundle them with something like Webpack and [https://github.com/zeit/pkg](https://github.com/zeit/pkg) in Node.js or comparable tools in other ecosystems.

### Apps

Apps for Grid are developed separately using Web technologies. While there are no restrictions and any UI Web framework or none at all could be used, we recommend React especially to be able to use [Ethereum React Components](https://github.com/ethereum/ethereum-react-components) and the hot reloading that is part of bootstrapping tools like [create-react-app](https://github.com/facebook/create-react-app).

To develop an app with Grid one needs to spin up a webserver for the application on port `3000` or point Grid's Webview to the correct location. **It is not recommended and will probably be disabled to navigate to hosted (non-localhost) websites or use Grid as a browser**. Grid is not a browser and is lacking security features that would allow safe browsing. **Only use Grid for your own apps and don't try to load other packages or websites.**

#### Grid API

The Grid API is currently very small, but powerful. Please expect breaking changes in the future though.

The globally injected Grid object has only one method:
```
const geth = await window.grid.getClient('geth')
```
This will query all available clients by their `name` property and return a reference to the client it can find.

From there you can interact with the client directly - please see the methods of the `PluginProxy` here:
[https://github.com/ethereum/grid/blob/master/ethereum_clients/Plugin.js#L167](https://github.com/ethereum/grid/blob/master/ethereum_clients/Plugin.js#L167)

The application can also subscribe to events like so:
```
const geth = await window.grid.getClient('geth')
geth.on('connected', () => {
  // use RPC methods here
})
```

The next section describes the Event types in more detail.


#### Events

Clients emit the following event types:

`starting`: the client binary is being executed in a separate process without feedback.

`started`: the client process is running and has notified Grid that it is ready.

`connected`: Grid has successfully established an IPC connection to the client. RPC API is available now.

`error`: an error occurred at any client lifecycle stage.

`stopped`: the client was stopped by the user or due to an event.

`log`: the client has written something to `stdout`, usually terminal output.

#### Example: Toy Wallet Starter

1. Create a new React app and start the server

```
$ npx create-react-app MyWallet
$ cd MyWallet
$ yarn
$ yarn start
```

> make sure that the server is running on port `3000`

2. Start Grid to see if the UI can be rendered

```
$ cd grid
$ yarn start:dev
```

3. Write code to interact with the Grid API

```
 $ yarn add ethereum-react-components
 $ yarn start

 delete App.css contents
 delete App.js body
 ```

 ```
import React, { Component } from 'react';
import './App.css';
import { AccountItem } from 'ethereum-react-components'

class App extends Component {

  state = {
    error: '',
    accounts: []
  }

  componentDidMount = async () => { 
    const geth = await window.grid.getClient('geth')
    if (geth) {
      try {
        // https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_accounts
        const accounts = await geth.sendRpc('eth_accounts')
        if (accounts) {
          console.log('accounts', accounts)
          this.setState({
            accounts: accounts
          })
        }
      } catch (err) {
        this.setState({
          error: err.message
        })
      }
    } else {
      this.setState({
        error: 'client not found'
      })
    }
  }
  renderAccountList = accounts => {
    // <div>{account} - idx</div>
    return (
      <div>
        { accounts.map((account, idx) => (
            <AccountItem name="Account 1" address={account} />
          ))}
      </div>
    )
  }
  render() {
    const { error, accounts } = this.state
    return (
      <div className="App">
        <h1>My Wallet - connected to Grid {window.grid.version}</h1>
        <div>
          {accounts && this.renderAccountList(accounts)}
        </div>
        <div>
          {error && ('error:' + error)}
        </div>
      </div>
    );  
  }
}

export default App;

 ```
