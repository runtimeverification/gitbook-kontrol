---
description: Take Kontrol of your smart contracts with Simple Formal Verification
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

# Kontrol

Struggling to kontrol your smart contracts behave as intended, even after thorough unit testing, fuzzing, and other validation procedures? Frustrated with finding bugs in your code but unable to prove their absence?

[**Kontrol**](https://github.com/runtimeverification/kontrol) is designed for you!


- **Open Source**: Kontrol is open source under the [BSD-3 Clause License](https://github.com/runtimeverification/kontrol/blob/master/LICENSE). This gives you full Kontrol to use, modify, and redistribute it according to your needs.
- **Easy to Use**: Kontrol gives you the advantage of using languages and testing frameworks you’re already familiar with. Kontrol enables developers to write formal specifications as [Foundry](https://book.getfoundry.sh/) tests in [Solidity](https://soliditylang.org). In addition, this allows developers to reuse their existing test suites for formal verification.
- **Scalable**: Kontrol automatically performs theorem proving on your specifications against your contracts through composable symbolic execution. It allows you to manipulate the proof process and provide simplification rules to achieve any proof goal. This ensures you can verify your contracts at any scale.
- **Trustworthy**: Kontrol has a trustworthy mathematical foundation for specifying and verifying smart contracts. It is built on the open-source, validated, and intuitive formal semantics of Ethereum Virtual Machine (EVM) bytecode, [KEVM](https://github.com/runtimeverification/evm-semantics). This provides an additional guarantee that the smart contracts you have verified will demonstrate the exact same behavior when executed by the virtual machine.

Even better, [KaaS](https://docs.runtimeverification.com/kaas) is a cloud-based solution that allows you to perform CI-Integrated formal verification on your contracts with speed and ease.
