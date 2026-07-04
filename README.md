5# Reentrancy-The-DAO-2016-60M-
What happened: The attacker's fallback function re-entered the withdraw() function before the DAO contract updated the caller's balance, draining funds in a loop.
Root cause: State mutation happened after the external call.
Fix: Update state before external calls (CEI pattern), and/or use OpenZeppelin's ReentrancyGuard. Prefer .transfer()/call with gas limits or pull-payment patterns for value transfers.
Base takeaway: Any Base contract handling withdrawals, claim() functions, or NFT mint refunds should default to CEI + nonReentrant. This is the single most preventable bug class in Solidity history — and it still shows up in 2024–2025 audits.
fix(vault): enforce checks-effects-interactions + add ReentrancyGuard

Ref: The DAO exploit (June 2016, ~$60M drained / 3.6M ETH)
Root cause: external call (msg.sender.call.value()) executed BEFORE
internal balance was set to zero, allowing recursive withdrawal.
Access Control — Parity Multisig Freeze (2017, $280M frozen)
fix(library): restrict initializer + remove public selfdestruct

Ref: Parity Multisig Wallet freeze (Nov 2017, ~$280M locked forever)
Root cause: shared library contract left uninitialized; anyone called
initWallet() to become "owner," then invoked selfdestruct() on the
library, bricking every wallet that delegatecalled into it.
What happened: A user accidentally became "owner" of the shared library contract and called selfdestruct, permanently freezing every wallet built on top of it.
Root cause: Missing access control on initialization + a delegatecall dependency on a mutable, killable library.
Fix: Never leave initializer functions unprotected. Avoid selfdestruct in shared/library contracts. Use OpenZeppelin's Initializable with _disableInitializers() in the constructor.
Base takeaway: If you're using proxy patterns (very common for Base apps to stay upgradeable), audit your implementation contract's constructor and initializer separately from the proxy — this exact bug class reappears every year in new clothes.
Oracle Manipulation — bZx Flash Loan Attacks (2020, ~$8M combined)
Root cause: Trusting a single, manipulable, single-block price source.
Fix: Use time-weighted average prices (TWAP), aggregate multiple independent oracles (e.g., Chainlink + a DEX TWAP), and add circuit breakers for abnormal price deltas.
Base takeaway: This is arguably THE flash-loan-era lesson. Any lending, perps, or liquidation logic on Base must never read price from a single spot pool it can be manipulated within the same transaction.
Cross-Chain Bridge — Poly Network (2021, $611M)
fix(bridge): restrict keeper role, add multi-sig verification on
cross-chain message execution

Ref: Poly Network exploit (Aug 2021, $611M — largest DeFi hack ever
at the time, later returned by the attacker)
Root cause: the contract that verified cross-chain messages trusted
a "keeper" address that could be changed by a spoofed cross-chain
call to itself — attacker changed the keeper to their own address,
then authorized unlimited withdrawals.
What happened: The attacker crafted a cross-chain message that tricked the bridge into changing its own trusted "keeper" role to an address they controlled, then drained assets across three chains.
Root cause: The privileged role-change function was reachable through the same generic message-execution path as normal user transactions — no separate, more restrictive authorization for changing trust roots.
Fix: Never let privileged/admin functions be reachable via generic external-call relays. Separate governance/role changes from user-facing message execution, and require multi-sig or timelock for any keeper/validator change.
Base takeaway: Bridges are the highest-value attack surface in crypto. If you're building or integrating a bridge into Base, the question isn't "can users call this" — it's "what OTHER paths can reach this function."
Signature Verification — Wormhole Bridge (2022, $325M)
fix(verify): require all guardian signatures to be validated against
the CURRENT signer set, reject legacy/unverified sysvar accounts

Ref: Wormhole bridge exploit (Feb 2022, $325M — 120,000 wrapped ETH)
Root cause: Solana program used a deprecated signature-verification
instruction that didn't actually confirm the signatures matched the
claimed guardian set, letting attacker forge a valid-looking mint
authorization.
