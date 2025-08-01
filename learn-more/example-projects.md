---
description: Code repositories, projects, and practical examples using Kontrol
---

# Example Projects

Here are notable projects and code repositories that use Kontrol for formal verification. These provide practical examples and templates for different use cases.

## Production Projects

### Optimism
[Optimism](https://github.com/ethereum-optimism/optimism/tree/c38ce096def52e4acdaecb7ffd2d396f464693dd/packages/contracts-bedrock/test/kontrol) uses Kontrol to verify critical components of their Bedrock contracts. Their setup includes:
- Custom deployment sequence for Kontrol proofs
- Automated proof generation and verification
- Integration with their CI pipeline
- Comprehensive documentation of their verification process

### Solady
[Kontrol-Solady](https://github.com/runtimeverification/kontrol-solady) is a formal verification project for the Solady library, demonstrating how to verify commonly used smart contract patterns and optimizations.

### Lido Dual Governance
[Lido's Dual Governance](https://github.com/lidofinance/dual-governance/tree/tests/kontrol-formal-verification/test/kontrol) uses Kontrol for formal verification of their dual governance mechanism.

## Educational Repositories

### Kontrol Demo
[Kontrol Demo](https://github.com/runtimeverification/kontrol-demo) is a simple repository demonstrating basic Kontrol usage, perfect for getting started with formal verification.

### Uniswap Hooks
[Uniswap Hooks Proofs](https://github.com/runtimeverification/uniswap-hooks) provides formal verification proofs for secure and modular Uniswap hooks. A detailed walkthrough of these proofs is available in the [Verified Hooks workshop video](https://www.youtube.com/watch?v=ubOfMnk0HZ0).

### Secureum Workshop
The [Secureum Kontrol Workshop](https://github.com/runtimeverification/secureum-kontrol) provides hands-on examples and exercises for learning Kontrol, including:
- Basic verification examples
- Advanced proof techniques
- Common patterns and best practices

### DSS 2024
[Kontrol DSS 2024](https://github.com/runtimeverification/kontrol-dss-2024) showcases more advanced usage patterns and verification techniques.

## Getting Started

To explore these examples:

1. Start with the [Kontrol Demo](https://github.com/runtimeverification/kontrol-demo) for basic usage
2. Review the [Secureum Workshop](https://github.com/runtimeverification/secureum-kontrol) for hands-on learning
3. Study the [Optimism implementation](https://github.com/ethereum-optimism/optimism/tree/c38ce096def52e4acdaecb7ffd2d396f464693dd/packages/contracts-bedrock/test/kontrol) for production usage
4. Explore [Kontrol-Solady](https://github.com/runtimeverification/kontrol-solady) for library verification patterns

Each project provides unique insights into how Kontrol can be used effectively in different contexts and for different purposes.

{% hint style="info" %}
**Looking for learning materials instead?** Check out our [Resources](resources.md) page for videos, blog posts, and educational content.
{% endhint %} 