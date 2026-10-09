# Severity classification

Every finding in this repository is rated from two inputs: **impact** and **likelihood**.

## Impact

| Level | Meaning |
| --- | --- |
| High | Loss or permanent freeze of user funds, or loss of control over the protocol |
| Medium | Limited or conditional loss of funds, or a core function that stops working |
| Low | No funds at risk; unexpected behaviour, degraded integration or user-error traps |

## Likelihood

| Level | Meaning |
| --- | --- |
| High | Any account can trigger it, at any time, at low cost |
| Medium | Requires specific conditions, a specific token or a specific market state |
| Low | Requires an unlikely configuration or a mistake by a privileged or careful actor |

## Matrix

| Impact \ Likelihood | High | Medium | Low |
| --- | --- | --- | --- |
| **High** | Critical | High | Medium |
| **Medium** | High | Medium | Low |
| **Low** | Medium | Low | Low |

**Informational** findings carry no direct risk: code quality, gas, readability, deviation from best practice.

## Finding format

Each finding contains:

- **ID and title**: `C-01`, `H-01`, `M-01`, `L-01`, `I-01`.
- **Location**: file, contract and function.
- **Description**: what the code does and which assumption it breaks.
- **Impact**: what an attacker or a user loses.
- **Recommendation**: a concrete change.

## Limits

Severity is a judgment, not a measurement. The same bug can be critical in a contract that holds funds and low in a contract that does not. When a rating depends on context, the report says so.
