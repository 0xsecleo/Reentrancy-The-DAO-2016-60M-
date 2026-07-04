# Reentrancy-The-DAO-2016-60M-
What happened: The attacker's fallback function re-entered the withdraw() function before the DAO contract updated the caller's balance, draining funds in a loop.
