---
description: Real-world projects and examples using Kontrol
---

# Example Projects

Here are some notable projects and examples that use Kontrol for formal verification:

## Production Projects

### Optimism
[Optimism](https://github.com/ethereum-optimism/optimism/tree/c38ce096def52e4acdaecb7ffd2d396f464693dd/packages/contracts-bedrock/test/kontrol) uses Kontrol to verify critical components of their Bedrock contracts. Their setup includes:
- Custom deployment sequence for Kontrol proofs
- Automated proof generation and verification
- Integration with their CI pipeline
- Comprehensive documentation of their verification process

### Solady
[Kontrol-Solady](https://github.com/runtimeverification/kontrol-solady) is a formal verification project for the Solady library, demonstrating how to verify commonly used smart contract patterns and optimizations.

## Educational Resources

### Patrick Collins' Course
[Cyfrin's Smart Contract Exploits Course](https://github.com/Cyfrin/sc-exploits-minimized/blob/main/test/invariant-break/formal-verification/KontrolTest.t.sol) includes Kontrol examples as part of their curriculum, showing how to use formal verification to prevent common vulnerabilities.

### Secureum Workshop
The [Secureum Kontrol Workshop](https://github.com/runtimeverification/secureum-kontrol) provides hands-on examples and exercises for learning Kontrol, including:
- Basic verification examples
- Advanced proof techniques
- Common patterns and best practices

## Example Repositories

### Kontrol Demo
[Kontrol Demo](https://github.com/runtimeverification/kontrol-demo) is a simple repository demonstrating basic Kontrol usage, perfect for getting started with formal verification.

### DSS 2024
[Kontrol DSS 2024](https://github.com/runtimeverification/kontrol-dss-2024) showcases more advanced usage patterns and verification techniques.

## Getting Started

To explore these examples:

1. Start with the [Kontrol Demo](https://github.com/runtimeverification/kontrol-demo) for basic usage
2. Review the [Secureum Workshop](https://github.com/runtimeverification/secureum-kontrol) for hands-on learning
3. Study the [Optimism implementation](https://github.com/ethereum-optimism/optimism/tree/c38ce096def52e4acdaecb7ffd2d396f464693dd/packages/contracts-bedrock/test/kontrol) for production usage
4. Explore [Kontrol-Solady](https://github.com/runtimeverification/kontrol-solady) for library verification patterns

Each project provides unique insights into how Kontrol can be used effectively in different contexts and for different purposes. 