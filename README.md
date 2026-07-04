# Reentrancy-The-DAO-2016-60M-
What happened: The attacker's fallback function re-entered the withdraw() function before the DAO contract updated the caller's balance, draining funds in a loop.
Root cause: State mutation happened after the external call.
Fix: Update state before external calls (CEI pattern), and/or use OpenZeppelin's ReentrancyGuard. Prefer .transfer()/call with gas limits or pull-payment patterns for value transfers.
Base takeaway: Any Base contract handling withdrawals, claim() functions, or NFT mint refunds should default to CEI + nonReentrant. This is the single most preventable bug class in Solidity history — and it still shows up in 2024–2025 audits.
