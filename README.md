# EthCapetownWorkshop

## What is Grid?

“Often referred to […] as “digital frontier", the Grid was made to provide an experimental platform where all forms of research could be carried out at unparalleled speeds.” - [Tron Movie](https://tron.fandom.com/wiki/Grid)

One way to describe Ethereum Grid is as a platform to run experiments, create prototypes (and hackathon projects) or develop fully working apps for the many available clients in the ecosystem. 
Another description could be that Ethereum Grid is the control center for all kinds of clients and Ethereum core binaries.

![Grid Screenshot](/assets/Grid-Screenshot.png)

With Grid, they can be downloaded, configured, and started all in one place. But even more than this, Grid serves as an Ethereum provider which means once a client is configured and started, DApps can connect to Grid and share the connection to the Ethereum network.

It is ideal for people who want to run a full node, have a convenient and secure way to update their binaries and don't want to rely on centralized 3rd party services like Infura / Metamask.

Grid can host and launch other interfaces and the functionality can be extended and adjusted through apps and plugins.

![Grid Tray Integration](/assets/Grid-Tray.png)


## Installation

Grid consists of 2 parts: the main application and the UI.
In order to hack on Grid we want to start it in `dev mode`which is why we clone both repos and run from source:

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

## Extending Grid

There are currently two ways to extends Grid's functionality or integrate with it.

1. Client plugins
2. Web Apps

Client plugins specify repositories of Ethereum clients and metadata to download and run them. The Grid can discover and load these plugin and will try to generate UI elements such as config forms for these clients.

Web apps are UI frontends written with Web technologies which can be hosted inside of Grid UI. They get access to the loaded clients and can interact through IPC with client processes and listen to state changes through events.

### Plugins

Grid uses a plugin architecture to integrate, query releases and interact with various Ethereum clients. To define a new client all you need to do is to add a new file to the [`ethereum_clients/client_plugins/`](https://github.com/ethereum/grid/tree/master/ethereum_clients/client_plugins) directory within Grid.
Grid will automatically detect the client and provide UI elements to manager releases for the specified client.

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

```

#### Plugin Properties

The `type` field is used to specify one of the possible types [`client`, `signer`, `storage`]. 

`order` is an optional field to specify a position where the client should be display within the UI.

The `displayName` is th name that is shown to the user while the `name` property is used internally to query and find certain plugins by name.

The `repository` field is a required information which is needed to discover and download different releases. 

`filter` options can be used to exclude or require certain keywords within the file name or version numbers and ranges that restrict the displayed download options.
`Filters` are applied after a release feed was downloaded which can be a heavy task. However, the filtering process can be sped up with `prefix` which uses server side API's to not even consider certain GitHub release assets or Azure hosted binaries as possible releases.

Grid (electron-app-manager) works on packages which are compressed container or archive files like `.zip` or `.tar`. The `binaryName` option specifies the name of the executable binary in the package. This is important because archives can contain more than one file, they can have nested sub-directories and binaries have platform specific extensions (.exe).

`resolveIpc` is used to defined a method that returns a valid IPC endpoint so that Grid can establish a secure connection to the spawned client process.

`flags` is an object that specifies the various command line arguments a client accepts which is used to generate the configuration view:

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


>Grid uses `electron-app-manager` under the hood which handles all the download and version management of the binaries so if some values are missing it can be helpful to check the docs of it as well.

#### Client Scripts

Some clients don't have executable binaries and require a runtime such as Python or Node.js (`nodex client.js` ). These clients are **currently not supported**. A way around this restriction is to bundle them with something like Webpack and [https://github.com/zeit/pkg](https://github.com/zeit/pkg) in Node.js or comparable tools in other ecosystems.

### Apps

Apps for Grid are developed separately using Web technologies. While there are no restrictions and any UI Web framework or non at all could be used, we recommend React especially to be able to use [Ethereum React Components](https://github.com/ethereum/ethereum-react-components) and the hot reloading that is part of bootstrapping tools like [create-react-app](https://github.com/facebook/create-react-app).

To develop an app with Grid one needs to spin up a webserver for the application on port `3000` or point Grid's Webview to the correct location. **It is not recommended and will probably be disabled to navigate to hosted (non-localhost) Websites or use Grid as a browser**. Grid is not a browser and is lacking security features that would allow safe browsing. Only use Grid for your own apps and don't try to load other packages or websites.

#### Grid API

```
```

#### Events

Clients emit the following event types:

`starting`: the client binary is being executed in a separate process without feedback.

`started`: the client process is running and has notified Grid that it is ready.

`connected`: Grid has successfully established an IPC connection to the client. RPC API is available now.

`error`: an error occurred at any client lifecycle stage.

`stopped`: the client was stopped by the user or due to an event.

`log`: the clientt has written something to `stdout` usually terminal output.

#### Example: Toy Wallet App

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