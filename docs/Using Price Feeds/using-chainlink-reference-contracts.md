---
layout: nodes.liquid
section: ethereum
date: Last Modified
title: "Introduction to Data Feeds"
permalink: "docs/using-chainlink-reference-contracts/"
whatsnext: {"Get the Latest Price":"/docs/get-the-latest-price/", "API Reference":"/docs/price-feeds-api-reference/", "Contract Addresses":"/docs/reference-contracts/"}
metadata:
  title: "Introduction to Data Feeds"
  description: "Add data to your smart contracts and applications. Chainlink data feeds include BTC/USD, BTC/ETH, ETH/USD and more!"
---
![Chainlink Abstract Banner](/files/2306b8b-Decentralized_Oracles_V3.png)

## Connect your contracts to the outside world

Chainlink Data Feeds are the quickest way to connect your smart contracts to the real-world data such as asset prices. One use for data feeds is to retrieve the latest pricing data of an asset in a single call and use that data either on-chain in a smart contract or off-chain in another application of your choice.

If you already have a project started and would like to integrate Chainlink, you can [add Chainlink to your existing project](../create-a-chainlinked-project/#install-into-existing-projects) by using the [`chainlink` NPM package](https://www.npmjs.com/package/@chainlink/contracts).

See the [Data Feeds Contract Addresses](/docs/reference-contracts/) page for a list of networks and proxy addresses. You can also use [data.chain.link](https://data.chain.link/) or the [Chainlink Market](https://market.link/) to select nodes for your requests.  

## Retrieve the latest asset prices

Often, smart contracts need to act in real-time on data such as prices of assets. This is especially true in [DeFi](https://defi.chain.link/).

For example, [Synthetix](https://www.synthetix.io/) uses Data Feeds to determine prices on their derivatives platform. Lending and borrowing platforms like [AAVE](https://aave.com/) use Data Feeds to ensure the total value of the collateral.

Data Feeds aggregate many data sources and publish them on-chain using a combination of the [Decentralized Data Model](/docs/architecture-decentralized-model/) and [Off-Chain Reporting](/docs/off-chain-reporting/).

## Components of a data feed

Data Feeds are an example of a decentralized oracle network and include the following components:

- **Consumer**: A consumer is an on-chain or off-chain application that uses Data Feeds. Consumer contracts use the [`AggregatorV3Interface`](https://github.com/smartcontractkit/chainlink/blob/master/contracts/src/v0.8/interfaces/AggregatorV3Interface.sol) to call functions on the proxy contract and retrieve information from the aggregator contract. For a complete list of functions available in the `AggregatorV3Interface`, see the [Data Feeds API Reference](/docs/price-feeds-api-reference/#aggregatorv3interface).
- **Proxy contract**: Proxy contracts are on-chain proxies that point to the aggregator for a particular data feed. Using proxies enables the underlying aggregator to be upgraded without any service interruption to consuming contracts. Proxy contracts can vary from one data feed to another, but the [`AggregatorProxy.sol` contract](https://github.com/smartcontractkit/chainlink/blob/develop/contracts/src/v0.7/dev/AggregatorProxy.sol) on Github is a common example.
- **Aggregator contract**: An aggregator is a contract that receives periodic data updates from the oracle network. Aggregators store aggregated data on-chain so that consumers can retrieve it and act upon it within the same transaction. For a complete list of functions and variables available on most aggregator contracts, see the [Data Feeds API Reference](/docs/price-feeds-api-reference/#accesscontrolledoffchainaggregator).

To learn how to create a consumer contract that uses an existing data feed, read the [Using Data Feeds](../get-the-latest-price/) documentation.

## Reading proxy and aggregator configurations

Because the proxy and aggregator contracts are all on-chain, you can see the current configuration by reading the variables through an [ABI](https://docs.soliditylang.org/en/latest/abi-spec.html) or using a blockchain explorer for your network. For example, you can see the [BTC/USD proxy configuration](https://etherscan.io/address/0xF4030086522a5bEEa4988F8cA5B36dbC97BeE88c#readContract) on the Ethereum network using Etherscan.

If you read the BTC/USD proxy configuration, you can query all of the functions and variables that are publicly accessible for that contract including the `aggregator` address, `latestRoundData()` function, `latestAnswer` variable, `owner` address, `latestTimestamp` variable, and several others. To see descriptions for the proxy contract variables and functions, see the source code for your specific data feed on [Etherscan](https://etherscan.io/address/0xF4030086522a5bEEa4988F8cA5B36dbC97BeE88c#code#L568).

The proxy contract points to an aggregator. This allows you to retrieve data through the proxy even if the aggregator is upgraded. If you view the `aggregator` address defined in the proxy configuration, you can see the aggregator and its configuration. For example, see the [BTC/USD aggregator contract](https://etherscan.io/address/0xF4030086522a5bEEa4988F8cA5B36dbC97BeE88c#code) in Etherscan. This contract includes several variables and functions, including another `latestRoundData()`. To see descriptions for the aggregator variables and functions, see the source code on [GitHub](https://github.com/smartcontractkit/libocr/blob/master/contract/AccessControlledOffchainAggregator.sol) or [Etherscan](https://etherscan.io/address/0xae74faa92cb67a95ebcab07358bc222e33a34da7#code#F1#L1).

You can call the `latestRoundData()` function directly on the aggregator, but it is a best practice to use the proxy instead so that changes to the aggregator do not affect your application. Similar to the proxy contract, the aggregator contract has a `latestAnswer` variable, `owner` address, `latestTimestamp` variable, and several others.

## Components of an aggregator

The aggregator contract has several variables and functions that might be useful for your application. Although aggregator contracts are similar for each data feed, some aggregators have different variables. Use the `typeAndVersion()` function on the aggregator to identify what type of aggregator it is and what version it is running.

Always check the contract source code and configuration to understand how specific data feeds operate. For example, the [aggregator contract for BTC/USD on Arbitrum](https://arbiscan.io/address/0x942d00008d658dbb40745bbec89a93c253f9b882#code) is different from the aggregators on other networks. The [contract on Kovan for BTC/USD](https://kovan.etherscan.io/address/0x222d3bd9bc8aef87afa9c8e4c7468da3f2c7130d#code) is also different from the contracts on other testnets and the Ethereum Mainnet.

For examples of the contracts that are typically used in aggregator deployments, see the [libocr repository](https://github.com/smartcontractkit/libocr/blob/master/contract/) on GitHub.

For a complete list of functions and variables available on most aggregator contracts, see the [Data Feeds API Reference](/docs/price-feeds-api-reference/#accesscontrolledoffchainaggregator).

## Updates to proxy and aggregator contracts

Aggregator contracts must be updated from time to time to add additional capabilities to data feeds. These updates include changes to the aggregator configuration or a complete replacement of the aggregator that the proxy uses. If you consume data feeds through the proxy, your applications can continue to operate during these changes.

Proxy and aggregator contracts all have an `owner` address that has permission to change variables and functions. For example, if you read the [BTC/USD proxy contract](https://etherscan.io/address/0xF4030086522a5bEEa4988F8cA5B36dbC97BeE88c#readContract) in Etherscan, you can see the `owner` address. This address is a [multi-signature safe](https://docs.gnosis-safe.io/introduction/the-programmable-account/gnosis-safe) (multisig) that you can also inspect.

If you [view the multisig contract](https://etherscan.io/address/0x21f73d42eb58ba49ddb685dc29d3bf5c0f0373ca#readProxyContract) in Etherscan using the _Read as Proxy_ feature, you can see the full details of the multisig including the list of addresses that can sign and the number of signers required for the multisig to approve actions on any contracts that it owns.

## Monitoring Data Feeds

When you build applications and protocols that depend on data feeds, include monitoring and safeguards to protect against the negative impact of extreme market events, possible malicious activity on third-party venues or contracts, potential delays, and outages.

Create your own monitoring alerts based on deviations in the answers that data feeds provide. This will notify you when potential issues occur so you can respond to them.

### Check the latest answer against reasonable limits

The data feed aggregator includes both [`minAnswer` and `maxAnswer` values](https://github.com/smartcontractkit/libocr/blob/9e4afd8896f365b964bdf769ca28f373a3fb0300/contract/AccessControlledOffchainAggregator.sol#L33). These variables prevent the aggregator from updating the `latestAnswer` outside the agreed range of acceptable values, but they do not stop your application from reading the most recent answer.

Configure your application to detect when the reported answer is close to reaching `minAnswer` or `maxAnswer` and issue an alert so you can respond to a potential market event. Separately, configure your application to detect and respond to extreme price volatility or prices that are outside of your acceptable limits.

### Check the timestamp of the latest answer

The aggregator updates its `latestAnswer` when the value deviates beyond a specified threshold or when the heartbeat idle time has passed. You can find the heartbeat and deviation values for each data feed at [data.chain.link](https://data.chain.link/) or in the [Contract Addresses](/docs/reference-contracts/) lists.

Your application should track the `latestTimestamp` variable or use the `updatedAt` value from the `latestRoundData()` function to make sure that the latest answer is recent enough for your application to use it. If your application detects that the reported answer is not updated within the heartbeat or within time limits that you determine are acceptable for your application, pause operation or switch to an alternate operation mode while identifying the cause of the delay.

During periods of low volatility, the heartbeat triggers updates to the latest answer. Some heartbeats are configured to last several hours, so your application should check the timestamp and verify that the latest answer is recent enough for your application.

To learn more about the heartbeat and deviation threshold, read the [Decentralized Data Model](/docs/architecture-decentralized-model/#aggregator) page.
