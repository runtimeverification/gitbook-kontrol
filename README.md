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
- **Easy to Use**: Take advantage of the languages and tests you’re already familiar with. Kontrol enables developers to write specifications in widely-used [Solidity](https://soliditylang.org). It seamlessly integrates with your existing [Foundry](https://book.getfoundry.sh/) test suites, allowing for simple formal verification of your contracts. Even if you're not a verification engineer, Kontrol lets you formally verify your contracts in minutes.
- **Trustworthy**: Kontrol has a trustworthy mathematical foundation for specifying and verifying smart contracts. It is built on the open-source, validated, and intuitive formal semantics of Ethereum Bytecode, [KEVM](https://github.com/runtimeverification/evm-semantics). Thus, Kontrol how your smart contracts execute on the virtual machine exactly as you formally verified them.
- **Scalable**: Kontrol automatically performs theorem proving on your specifications against your contracts through composable symbolic execution. It allows you to manipulate the proof process and provide simplification rules to achieve any proof goal. This ensures you can verify your contracts at any scale.


