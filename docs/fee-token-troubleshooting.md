# Fee Token Troubleshooting (Tempo Moderato)

## Symptom: simulate OK, send FAIL (“Internal JSON-RPC error”)
You may see a `simulate` succeed, but transaction submission fails with a generic wallet/provider error.

In our case, the underlying node rejection (visible via RPC error details) was:
- “Insufficient liquidity for fee token …”

## Root cause: active proposer/validator preferred token
Fee liquidity checks depend on the *active proposer/validator’s preferred token*.
This can change per block.

Practical implication:
- Having liquidity in `Pool(userToken, AlphaUSD)` may still fail if the current proposer prefers `PathUSD`.
- The required pool is `Pool(userToken, proposerPreferredToken)`.

## Quick debug steps
1) Read latest block proposer address:
   - read latest block `coinbase` (or `miner` depending on RPC)
2) Read preferred validator token:
   - call `FeeManager.validatorTokens(coinbase)`
3) Ensure Fee AMM liquidity exists for:
   - `Pool(userToken, preferredValidatorToken)`
   - specifically: `reserveValidatorToken > 0`

## Fix
Add validator-token liquidity to the pool against the proposer preferred token (often PathUSD on Moderato), then retry:
- `FeeManager.setUserToken(userToken)`

## Notes
- `quoteToken` (DEX pricing/routing) does not need to match the validator preferred token.
- Explorer is the source of truth for which fee token was used; wallet UIs may show a generic chain fee token.
