---
description: Formally verify smart contracts using the tools you already know
layout:
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Kontrol

<div data-full-width="true">

<figure><img src=".gitbook/assets/kontrol logo yellow.png" alt=""><figcaption></figcaption></figure>

</div>

[**Kontrol**](https://github.com/runtimeverification/kontrol) combines **KEVM** and [Foundry](https://book.getfoundry.sh/) to grant developers the ability to perform [formal verification](https://en.wikipedia.org/wiki/Formal\_verification) without learning a new language or tool. This is especially useful for those who are not verification engineers. Additionally, developers can leverage Foundry test suites they have already developed and use symbolic execution to increase the level of confidence.

[**KEVM**](https://github.com/runtimeverification/evm-semantics) is a tool that enables formal verification of smart contracts on the [Ethereum](https://ethereum.org/en/) blockchain. It provides a mathematical foundation for specifying and implementing smart contracts. Developers can use **KEVM** to rigorously reason about the behavior of their smart contracts, ensuring correctness and reducing the likelihood of vulnerabilities in the contract code.

[Foundry](https://book.getfoundry.sh/) is a smart contract development toolchain. It manages dependencies, compiles projects, runs tests, facilitates deployments and provides a command-line interface to interact with the chain via [Solidity](https://soliditylang.org/) scripts. If you’re curious about Foundry, have a look at one of our blog posts about it [here](https://runtimeverification.com/blog/foundry-gen-2-of-ethereum-tooling).
