---
layout: ../../../../layouts/MainLayout.astro
section: nodeOperator
date: Last Modified
title: "Off-chain Reporting Jobs"
permalink: "docs/jobs/types/offchain-reporting/"
---

OCR jobs (off-chain reporting jobs) are used very similarly to Flux Monitor jobs. They update data feeds with aggregated data from many Chainlink oracle nodes. However, they do this aggregation using a cryptographically-secure off-chain protocol that makes it possible for only a single node to submit all answers from all participating nodes during each round (with proofs that the other nodes' answers were legitimately provided by those nodes), which saves a significant amount of gas.

Off-chain reporting jobs require the `FEATURE_OFFCHAIN_REPORTING=true` environment variable.

#### Bootstrap node

Every OCR cluster requires at least one bootstrap node as a kind of "rallying point" that enables the other nodes to find one another. Bootstrap nodes do not participate in the aggregation protocol and do not submit answers to the feed.

**Spec format**

```toml
type               = "offchainreporting"
schemaVersion      = 1
evmChainID         = 1
contractAddress    = "0x27548a32b9aD5D64c5945EaE9Da5337bc3169D15"
p2pBootstrapPeers  = [
    "/dns4/chain.link/tcp/1234/p2p/16Uiu2HAm58SP7UL8zsnpeuwHfytLocaqgnyaYKP8wu7qRdrixLju",
]
isBootstrapPeer = true
externalJobID   = "0EEC7E1D-D0D2-476C-A1A8-72DFB6633F05"
```

**Shared fields**
See [shared fields](/chainlink-nodes/oracle-jobs/jobs/#shared-fields).

**Unique fields**

- `contractAddress`: The address of the `OffchainReportingAggregator` contract.
- `evmChainID`: The chain ID of the EVM chain in which the job will operate.
- `p2pBootstrapPeers`: A list of libp2p dial addresses of the other bootstrap nodes helping oracle nodes find one another on the network. It is used with P2P networking stack V1 as follows: `p2pBootstrapPeers  = [ "/dns4/<host name or ip>/tcp/<port>/p2p/<bootstrap node's P2P ID>" ]`
- `p2pv2Boostrappers`: A list of libp2p dial addresses of the other bootstrap nodes helping oracle nodes find one another on the network. It is used with P2P networking stack V2 as follows: `p2pv2Boostrappers = [ "<bootstrap node's P2P ID>@<host name or ip>:<port>" ]`
- `isBootstrapPeer`: This must be set to `true`.

**Job type specific pipeline variables**

- `$(jobSpec.databaseID)`: The ID of the job spec in the local database. You shouldn't need this in 99% of cases.
- `$(jobSpec.externalJobID)`: The globally-unique job ID for this job. Used to coordinate between node operators in certain cases.
- `$(jobSpec.name)`: The local name of the job.
- `$(jobRun.meta)`: A map of metadata that can be sent to a bridge, etc.

#### Oracle node

Oracle nodes, on the other hand, are responsible for submitting answers.

**Spec format**

```toml
type               = "offchainreporting"
schemaVersion      = 1
evmChainID         = 1
name               = "OCR: ETH/USD"
contractAddress    = "0x613a38AC1659769640aaE063C651F48E0250454C"
externalJobID      = "0EEC7E1D-D0D2-476C-A1A8-72DFB6633F06"
p2pPeerID          = "12D3KooWApUJaQB2saFjyEUfq6BmysnsSnhLnY5CF9tURYVKgoXK"
p2pBootstrapPeers  = [
    "/dns4/chain.link/tcp/1234/p2p/16Uiu2HAm58SP7UL8zsnpeuwHfytLocaqgnyaYKP8wu7qRdrixLju",
]
isBootstrapPeer    = false
keyBundleID        = "7f993fb701b3410b1f6e8d4d93a7462754d24609b9b31a4fe64a0cb475a4d934"
monitoringEndpoint = "chain.link:4321"
transmitterAddress = "0xF67D0290337bca0847005C7ffD1BC75BA9AAE6e4"
observationTimeout = "10s"
blockchainTimeout  = "20s"
contractConfigTrackerSubscribeInterval = "2m"
contractConfigTrackerPollInterval = "1m"
contractConfigConfirmations = 3
observationSource = """
    // data source 1
    ds1          [type="bridge" name=eth_usd]
    ds1_parse    [type="jsonparse" path="one,two"]
    ds1_multiply [type="multiply" times=100]

    // data source 2
    ds2          [type="http" method=GET url="https://chain.link/eth_usd"
                  requestData="{\\"hi\\": \\"hello\\"}"]
    ds2_parse    [type="jsonparse" path="three,four"]
    ds2_multiply [type="multiply" times=100]

    ds1 -> ds1_parse -> ds1_multiply -> answer
    ds2 -> ds2_parse -> ds2_multiply -> answer

    answer [type=median]
"""
```

**Shared fields**
See [shared fields](/chainlink-nodes/oracle-jobs/jobs/#shared-fields).

**Unique fields**

- `contractAddress`: The address of the `OffchainReportingAggregator` contract.
- `evmChainID`: The chain ID of the EVM chain in which the job will operate.
- `p2pPeerID`: The base58-encoded libp2p public key of this node.
- `p2pBootstrapPeers`: A list of libp2p dial addresses of the other bootstrap nodes helping oracle nodes find one another on the network. It is used with P2P networking stack V1 as follows: `p2pBootstrapPeers  = [ "/dns4/<host name or ip>/tcp/<port>/p2p/<bootstrap node's P2P ID>" ]`
- `p2pv2Boostrappers`: A list of libp2p dial addresses of the other bootstrap nodes helping oracle nodes find one another on the network. It is used with P2P networking stack V2 as follows: `p2pv2Boostrappers = [ "<bootstrap node's P2P ID>@<host name or ip>:<port>" ]`
- `keyBundleID`: The hash of the OCR key bundle to be used by this node. The Chainlink node keystore manages these key bundles. Use the node **Key Management** UI or the `chainlink keys ocr` sub-commands in the CLI to create and manage key bundles.
- `monitoringEndpoint`: The URL of the telemetry endpoint to send OCR metrics to.
- `transmitterAddress`: The Ethereum address from which to send aggregated submissions to the OCR contract.
- `observationTimeout`: The maximum duration to wait before an off-chain request for data is considered to be failed/unfulfillable.
- `blockchainTimeout`: The maximum duration to wait before an on-chain request for data is considered to be failed/unfulfillable.
- `contractConfigTrackerSubscribeInterval`: The interval at which to retry subscribing to on-chain config changes if a subscription has not yet successfully been made.
- `contractConfigTrackerPollInterval`: The interval at which to proactively poll the on-chain config for changes.
- `contractConfigConfirmations`: The number of blocks to wait after an on-chain config change before considering it worthy of acting upon.

**Job type specific pipeline variables**

- `$(jobSpec.databaseID)`: The ID of the job spec in the local database. You shouldn't need this in 99% of cases.
- `$(jobSpec.externalJobID)`: The globally-unique job ID for this job. Used to coordinate between node operators in certain cases.
- `$(jobSpec.name)`: The local name of the job.
- `$(jobRun.meta)`: A map of metadata that can be sent to a bridge, etc.
