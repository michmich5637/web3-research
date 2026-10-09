# web3-research

Research notes and smart contract security reviews, focused on the EVM and Layer 2 ecosystems.

Everything here is written to be checked: each claim points to the code or the source it comes from, and each finding states its impact, its likelihood and a concrete fix.

## Contents

| Folder | What it contains |
| --- | --- |
| [`security/`](./security) | Security reviews of smart contracts |
| [`methodology/`](./methodology) | How findings are classified and reported |
| [`templates/`](./templates) | Reusable report templates |

## Security reviews

| Date | Target | Scope | Findings | Report |
| --- | --- | --- | --- | --- |
| 2026-10 | `michmich56` ERC-20 token | 1 contract | 2 critical, 1 low, 3 informational | [Report](./security/2026-10-michmich56-erc20.md) |
| 2026-10 | `SimpleTokenSwap` (Uniswap V3 wrapper) | 1 contract | 2 medium, 3 low, 4 informational | [Report](./security/2026-10-simple-token-swap.md) |

Both reviews cover my own early contracts. Reviewing code I wrote is a deliberate starting point: the findings are real, the fixes are tracked in the corresponding repositories, and the process is the same one I apply to third-party code.

## Method

1. Read the code and its dependencies in full before forming an opinion.
2. Write down the trust assumptions: who can call what, and with which funds.
3. Look for broken assumptions: access control, token handling, external calls, arithmetic, integration specifics.
4. Classify each finding with the [severity matrix](./methodology/severity-classification.md).
5. Recommend a fix that can be implemented and tested.

## Disclaimer

These documents are provided for educational and informational purposes. A review is a time-boxed assessment and does not guarantee the absence of vulnerabilities. Nothing here is financial advice.

## License

Text and reports are released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
