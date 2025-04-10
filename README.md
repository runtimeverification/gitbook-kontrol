---
description: Kontrol your smart contracts with formal verification made simple
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

<div data-full-width="true">

<figure><img src=".gitbook/assets/kontrol logo yellow.png" alt=""><figcaption></figcaption></figure>

</div>

[**Kontrol**](https://github.com/runtimeverification/kontrol) is a powerful formal verification tool for EVM smart contracts that makes complex verification easy. Kontrol seamlessly integrates with Foundry's testing framework, allowing developers to utilize existing Foundry tests as formal specifications. This approach simplifies the verification process, making it accessible even to those without a background in formal verification.

- **Open Source**: Kontrol is open source under the [BSD-3 Clause License](https://github.com/runtimeverification/kontrol/blob/master/LICENSE). This gives you full kontrol to use, modify, and redistribute it according to your needs.
- **User Friendly**: Kontrol gives you the advantage of using languages and testing frameworks you're already familiar with. Kontrol enables developers to write formal specifications as [Foundry](https://book.getfoundry.sh/) tests in [Solidity](https://soliditylang.org). In addition, this allows developers to reuse their existing test suites for formal verification.
- **Scalable**: Kontrol automatically verifies your contracts against your formal specifications via compositional symbolic execution. It generates proofs during the verification process, allowing you to verify contracts of any size. Additionally, Kontrol enhances efficiency by supporting lemmas, loop invariants, and bounded model checking.
- **Trustworthy**: Kontrol has a trustworthy mathematical foundation for specifying and verifying smart contracts. It is built on the open-source, validated, and intuitive formal semantics of Ethereum Virtual Machine (EVM) bytecode, [KEVM](https://github.com/runtimeverification/evm-semantics). This provides an additional guarantee that the smart contracts you have verified will demonstrate the exact same behavior when executed by the virtual machine.
- **CI Integration**: Kontrol seamlessly integrates into continuous integration (CI) pipelines, enabling automated verification with each code commit. Through [Kontrol as a Service (KaaS)](https://docs.runtimeverification.com/kaas), it offers cloud-based verification and visualization features, ensuring continuous assurance of smart contract integrity.

## Getting Started

To begin using Kontrol, check out our [installation guide](overview/readme/installations.md) and explore our [example projects](guides/kontrol-example/README.md) to see Kontrol in action.

## Community

Join our growing community on [Discord](https://discord.gg/CurfmXNtbN) to connect with other developers, share experiences, and get support.
