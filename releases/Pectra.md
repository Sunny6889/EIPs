# The Ethereum Prague-Electra Upgrade

The Ethereum Prague-Electra upgrade, commonly referred to as the Pectra upgrade, went live on the Ethereum mainnet on May 7, 2025. It combines the Prague (execution layer) and Electra (consensus layer) upgrades and includes 11 Ethereum Improvement Proposals (EIPs) to enhance scalability, user experience, staking efficiency, and network performance.

## List of EIPs in the Pectra (Prague-Electra) Upgrade

The following EIPs are confirmed to be part of the Pectra upgrade, as referenced across multiple sources.

- **EIP-2537**: Precompile for BLS12-381 curve operations
- **EIP-2935**: Save historical block hashes in state
- **EIP-6110**: Supply validator deposits on-chain
- **EIP-7002**: Execution layer triggerable exits
- **EIP-7251**: Increase the MAX_EFFECTIVE_BALANCE @Jiao Sunny
- **EIP-7549**: Move committee index outside Attestation @Jiao Sunny
- **EIP-7685**: General purpose execution layer requests
- **EIP-7702**: Set EOA account code
- **EIP-7742**: Blob count configuration
- **EIP-7762**: Increase blob target and max
- **EIP-7691**: EOF - EVM Object Format (likely included, though some sources note it as considered)

*Note*: No Ethereum Request for Comments (ERCs) are explicitly listed in the Pectra upgrade. ERCs typically define application-layer standards (e.g., ERC-20 for tokens), while EIPs in Pectra focus on core protocol, networking, and execution layer improvements. If ERCs were intended in the query, please clarify, and I can provide further analysis.

## Classification of EIPs by Direction

The EIPs are classified based on their primary focus or "direction" within the Ethereum ecosystem, such as improving staking, user experience, scalability, or network efficiency. The directions are derived from the goals outlined in sources, including enhancing validator operations, account abstraction, Layer 2 scalability, and protocol efficiency.

### 1. Staking and Validator Operations

These EIPs aim to optimize the staking process, improve validator flexibility, and streamline operations for Ethereum’s Proof-of-Stake consensus.

- **EIP-6110**: Supply validator deposits on-chain @blade.han
  - **Description**: Moves validator deposit processing to the execution layer, embedding deposits in the block structure to reduce onboarding delays from ~9 hours to ~13 minutes and enhance security.
  - **Impact**: Simplifies validator onboarding, reduces complexity, and improves transparency for stakers.

- **EIP-7002**: Execution layer triggerable exits @blade.han
  - **Description**: Allows validators to trigger exits and partial withdrawals directly from the execution layer using withdrawal credentials, enabling smart contract-based staking management.
  - **Impact**: Enhances flexibility for validators, supports automated staking services, and reduces reliance on consensus layer mechanisms.

- **EIP-7251**: Increase the MAX_EFFECTIVE_BALANCE @Jiao Sunny
  - **Description**: Raises the maximum validator balance from 32 ETH to 2,048 ETH, allowing validators to stake larger amounts and consolidate stakes into fewer validators.
  - **Impact**: Reduces validator count, lowers network communication load, and enables compound interest for stakers (e.g., staking 42 ETH instead of splitting into multiple 32 ETH validators).

### 2. User Experience and Account Abstraction

These EIPs focus on improving how users interact with Ethereum, particularly through wallet functionality and account abstraction, making transactions more seamless and flexible.

- **EIP-7702**: Set EOA account code @blade.han @Zeus.J
  - **Description**: Introduces a new transaction type (SET_CODE_TX_TYPE=0x04) that allows Externally Owned Accounts (EOAs) to temporarily execute smart contract code during transactions, enabling features like batch transactions, gas sponsorship, and passkey-based authentication.
  - **Impact**: Bridges EOAs and smart contract wallets, enhancing wallet usability with features like transaction batching and social recovery. Replaces EIP-3074 for better compatibility with ERC-4337 account abstraction.

*Note*: EIP-3074 was initially considered but replaced by EIP-7702 due to security concerns and better alignment with account abstraction goals.

### 3. Layer 2 Scalability and Data Availability

These EIPs enhance Ethereum’s rollup-centric scaling strategy by increasing blob capacity and optimizing data availability for Layer 2 solutions.

- **EIP-7742**: Blob count configuration @Federico
  - **Description**: Enables dynamic adjustments of blob target and maximum counts via a configuration file, with a parameter (baseFeeUpdateFraction) to adjust blob gas pricing responsiveness.
  - **Impact**: Provides flexibility for future blob scaling, supporting Layer 2 rollups by optimizing data pricing and availability.

- **EIP-7762**: Increase blob target and max @Federico
  - **Description**: Increases the target blob count per block from 3 to 6 and the maximum from 6 to 9, expanding data capacity for Layer 2 rollups.
  - **Impact**: Reduces transaction costs and congestion for Layer 2 solutions, supporting Ethereum’s scalability roadmap.

### 4. Network Efficiency and Security

These EIPs improve Ethereum’s performance, security, and statelessness by optimizing cryptographic operations, data storage, and consensus mechanisms.

- **EIP-2537**: Precompile for BLS12-381 curve operations @Federico
  - **Description**: Introduces precompiled contracts for BLS12-381 elliptic curve operations, enabling efficient BLS signature verification and zero-knowledge proofs.
  - **Impact**: Reduces computational costs for Layer 2 solutions and cross-chain bridges, enhancing security and efficiency for multi-party computations.

- **EIP-2935**: Save historical block hashes in state @Zeus.J
  - **Description**: Stores the last 8,192 block hashes (~27 hours) in a system contract, enabling smart contracts to query historical data and supporting stateless clients.
  - **Impact**: Reduces storage requirements for nodes, lays groundwork for statelessness, and improves reliability for applications needing historical data.

- **EIP-7549**: [refer](../EIPS/eip-7549.md) Move committee index outside Attestation @Jiao Sunny
  - **Description**: Removes the committee index from the signature message in the consensus mechanism, allowing more efficient vote aggregation.
  - **Impact**: Lowers network load and verification costs, improving consensus layer efficiency.

- **EIP-7685**: General purpose execution layer requests @tina.lee
  - **Description**: Introduces a framework for direct communication between execution and consensus layers, streamlining operations like deposits, withdrawals, and validator consolidations.
  - **Impact**: Enhances inter-layer efficiency, enables smart contracts to interact with the consensus layer, and reduces operational complexity.

### 5. Smart Contract and EVM Enhancements

This EIP focuses on improving the Ethereum Virtual Machine (EVM) and smart contract deployment efficiency.

- **EIP-7691**: EOF - EVM Object Format @Federico
  - **Description**: Introduces a new format for EVM bytecode to optimize smart contract deployment and execution, potentially reducing gas costs.
  - **Impact**: Enhances smart contract performance and developer experience, though its inclusion is noted as tentative in some sources due to complexity.

## Summary Table

| Direction                             | EIPs                                   | Focus                                                                 |
|---------------------------------------|----------------------------------------|----------------------------------------------------------------------|
| Staking and Validator Operations      | EIP-6110, EIP-7002, EIP-7251           | Streamline staking, improve validator flexibility, reduce network load |
| User Experience and Account Abstraction | EIP-7702                               | Enhance wallet functionality, enable batch transactions and gas sponsorship |
| Layer 2 Scalability and Data Availability | EIP-7742, EIP-7762                     | Increase blob capacity, optimize data availability for Layer 2 rollups |
| Network Efficiency and Security       | EIP-2537, EIP-2935, EIP-7549, EIP-7685 | Improve cryptography, statelessness, consensus efficiency, and inter-layer communication |
| Smart Contract and EVM Enhancements   | EIP-7691                               | Optimize EVM bytecode and smart contract deployment                    |

## Notes and Clarifications

- **No ERCs Identified**: The Pectra upgrade focuses on core protocol EIPs rather than application-layer ERCs. If the query intended to include ERCs (e.g., ERC-4337 for account abstraction), note that ERC-4337 is not part of Pectra but is complemented by EIP-7702.
- **EIP-3074 Replacement**: EIP-3074 was initially proposed but replaced by EIP-7702 due to security concerns and better alignment with account abstraction goals.
- **EIP-7691 (EOF) Status**: Some sources list EIP-7691 as tentative or considered, as it was part of discussions but not always confirmed in final lists. Its inclusion is assumed here based on its mention in upgrade analyses.

## Additional Information

- **Upgrade Context**: Pectra is the most feature-packed Ethereum upgrade to date, following the Dencun upgrade in March 2024. It aims to fix network issues, improve user experience, and prepare for future upgrades like Fusaka.
- **Timeline**: The upgrade was tested on the Sepolia testnet on March 5, 2025, and activated on the mainnet on May 7, 2025.
- **Impact**: The upgrade enhances Ethereum’s scalability, reduces transaction costs for Layer 2 solutions, simplifies staking, and improves wallet usability, positioning Ethereum for broader adoption.

