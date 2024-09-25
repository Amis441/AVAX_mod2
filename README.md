
# Avalanche HyperSDK: Create a Custom Subnet

## Project Overview

This project demonstrates how to use the Avalanche HyperSDK to create a custom virtual machine and subnet on the Avalanche platform. The HyperSDK allows developers to build and customize blockchains tailored to specific use cases, such as token minting, transfer, and asset trading.

## Features

- **Custom Virtual Machine**: Build a blockchain with tailored functionality using HyperSDK.
- **Token Operations**: Define rules for creating, minting, and transferring tokens.
- **Order Book Management**: Implement an order book for asset trading.

## Challenge

Your startup has identified a need to create a custom virtual machine to enable users to mint and transfer tokens. The HyperSDK provides a powerful solution for this task by allowing you to build a custom blockchain tailored to your specific needs. With the HyperSDK, you can define the rules and functionality of your chain, including the ability to create and transfer tokens and manage order books for trading assets.

## Disclaimer

Since HyperSDK is alpha software, we are using the example token VM, which is one of the few that has been tested and guaranteed to be maintained in the upcoming future. Consult the repo constantly to keep your project up to date.

## Getting Started

### Prerequisites

Before you begin, ensure you have Go installed on your system. If not, you can download and install it from [here](https://golang.org/dl/).

### Installation

1. **Clone the Repository:**

   ```bash
   git clone <repository_url>
   cd <repository_directory>

2. **Normalize Dependencies:**

   ```bash
   go mod tidy

### Configure Project Constants:


Edit `consts/consts.go` (same as below) to define the constants and initialize necessary values for your project.

```go
// Copyright (C) 2023, Ava Labs, Inc. All rights reserved.
// See the file LICENSE for licensing terms.

package consts

import (
	"github.com/ava-labs/avalanchego/ids"
	"github.com/ava-labs/avalanchego/vms/platformvm/warp"
	"github.com/ava-labs/hypersdk/chain"
	"github.com/ava-labs/hypersdk/codec"
	"github.com/ava-labs/hypersdk/consts"
)

const (
	// TODO: choose a human-readable part for your hyperchain
	HRP = "AyushPanwar"
	// TODO: choose a name for your hyperchain
	Name = "Ayush"
	// TODO: choose a token symbol
	Symbol = "AP"
)

var ID ids.ID

func init() {
	b := make([]byte, consts.IDLen)
	copy(b, []byte(Name))
	vmID, err := ids.ToID(b)
	if err != nil {
		panic(err)
	}
	ID = vmID
}

// Instantiate registry here so it can be imported by any package. We set these
// values in [controller/registry].
var (
	ActionRegistry *codec.TypeParser[chain.Action, *warp.Message, bool]
	AuthRegistry   *codec.TypeParser[chain.Auth, *warp.Message, bool]
)
```
### Register Actions:


Edit `registry/registry.go` (same as below) to register actions and initialize registries for your custom blockchain project.

```go
// Copyright (C) 2023, Ava Labs, Inc. All rights reserved.
// See the file LICENSE for licensing terms.

package registry

import (
	"github.com/ava-labs/avalanchego/utils/wrappers"
	"github.com/ava-labs/avalanchego/vms/platformvm/warp"
	"github.com/ava-labs/hypersdk/chain"
	"github.com/ava-labs/hypersdk/codec"

	"tokenvm/actions"
	"tokenvm/auth"
	"tokenvm/consts"
)

// Setup types
func init() {
	consts.ActionRegistry = codec.NewTypeParser[chain.Action, *warp.Message]()
	consts.AuthRegistry = codec.NewTypeParser[chain.Auth, *warp.Message]()

	errs := &wrappers.Errs{}
	errs.Add(
		// When registering new actions, ALWAYS make sure to append at the end.
		consts.ActionRegistry.Register(&actions.Transfer{}, actions.UnmarshalTransfer, false),

		//register action: actions.CreateAsset
		//register action: actions.MintAsset
		consts.ActionRegistry.Register(&actions.CreateAsset{}, actions.UnmarshalBurnAsset, false),
		consts.ActionRegistry.Register(&actions.MintAsset{}, actions.UnmarshalModifyAsset, false),

		consts.ActionRegistry.Register(&actions.BurnAsset{}, actions.UnmarshalBurnAsset, false),
		consts.ActionRegistry.Register(&actions.ModifyAsset{}, actions.UnmarshalModifyAsset, false),

		consts.ActionRegistry.Register(&actions.CreateOrder{}, actions.UnmarshalCreateOrder, false),
		consts.ActionRegistry.Register(&actions.FillOrder{}, actions.UnmarshalFillOrder, false),
		consts.ActionRegistry.Register(&actions.CloseOrder{}, actions.UnmarshalCloseOrder, false),

		consts.ActionRegistry.Register(&actions.ImportAsset{}, actions.UnmarshalImportAsset, true),
		consts.ActionRegistry.Register(&actions.ExportAsset{}, actions.UnmarshalExportAsset, false),

		// When registering new auth, ALWAYS make sure to append at the end.
		consts.AuthRegistry.Register(&auth.ED25519{}, auth.UnmarshalED25519, false),
	)
	if errs.Errored() {
		panic(errs.Err)
	}
}

```
### Run the Virtual Machine Locally:

Ensure Go is on your path. If not, run:

```bash
export PATH=$PATH:$(go env GOPATH)/bin
```
### Execute the following commands to run and build the VM:

```bash
MODE="run-single" ./scripts/run.sh
./scripts/build.sh
```
### Load Demo Private Key:

Import the demo private key included in the project:

```bash
./build/token-cli key import demo.pk
./build/token-cli chain import-anr
```
### Interact with Your HyperChain:

### Mint and Trade
#### Step 1: Create Your Asset
First up, let's create our own asset. You can do so by running the following
command from this location:
```bash
./build/token-cli action create-asset
```

When you are done, the output should look something like this:
```
database: .token-cli
address: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp
chainID: 2qBqBsx7VyNCSeSHw8vPgHK4iFtLBzrCppsVQEE4YRmtVBpBXF
metadata (can be changed later): myAssest
continue (y/n): y
✅ txID: SP56Fs9vE2YP9kwLM5hQSVGJvqEY9ii71zzdoR3rgx4AppABuN
```
_`txID` is the `assetID` of your new asset._

The "loaded address" here is the address of the default private key (`demo.pk`). We
use this key to authenticate all interactions with the `tokenvm`.

#### Step 2: Mint Your Asset

After we've created our own asset, we can now mint some of it. You can do so by
running the following command from this location:
```bash
./build/token-cli action mint-asset
```

When you are done, the output should look something like this (usually easiest
just to mint to yourself).
```
database: .token-cli
address: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp
chainID: 2qBqBsx7VyNCSeSHw8vPgHK4iFtLBzrCppsVQEE4YRmtVBpBXF
assetID: BcJeaMTZn7k8Ec2pwx785aV6Q8jq4KHJ9Ti8RncVq5GpFAcRK
metadata: myAssest supply: 0
recipient: token1rvzhmceq997zntgvravfagsks6w0ryud3rylh4cdvayry0dl97nsjzf3yp
amount: 2000
continue (y/n): y
✅ txID: BcJeaMTZn7k8Ec2pwx785aV6Q8jq4KHJ9Ti8RncVq5GpFAcRK
```

To shut down the local Avalanche network, run:

```bash
killall avalanche-network-runner

```
### CONCLUSION
- So we have created a custom virtual machine to enable users to mint and transfer tokens.
- With the HyperSDK, we have defined the rules and functionality of our chain, including the ability to create, mint and transfer tokens .

