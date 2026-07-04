# Reentrancy-The-DAO-2016-60M-
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
