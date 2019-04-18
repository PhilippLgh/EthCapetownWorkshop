# EthCapetownWorkshop

## What is Grid?

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

## Plugin structure

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

>Grid uses `electron-app-manager` under the hood which handles all the download and version management of the binaries so if some values are missing it can be helpful to check the docs of it as well.

### Plugin Properties

The `type` field is used to specify one of the possible types [`client`, `signer`, `storage`]. `order` is an optional field to specify a position where the client should be display within the UI.

The `displayName` is th name that is shown to the user while the `name` property is used internally to query and find certain plugins by name.

The `repository` field is a required information which is needed to discover and download different releases. 

`filter` options can be used to exclude or require certain keywords within the file name or version numbers and ranges that restrict the displayed download options.
`Filters` are applied after a release feed was downloaded which can be a heavy task. However, the filtering process can be sped up with `prefix` which uses server side API's to not even consider certain GitHub release assets or Azure hosted binaries as possible releases.

Grid (electron-app-manager) works on packages which are compressed container or archive files like `.zip` or `.tar`. The `binaryName` option specifies the name of the executable binary in the package. This is important because archives can contain more than one file, they can have nested sub-directories and binaries have platform specific extensions (.exe).

### Client Scripts

Some clients don't have executable binaries and require a runtime such as Python or Node.js (`nodex client.js` ). These clients are **currently not supported**. A way around this restriction is to bundle them with something like Webpack and [https://github.com/zeit/pkg](https://github.com/zeit/pkg) in Node.js and comparable tools in other ecosystems.