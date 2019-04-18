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