---
name: tempoclaw-mpp-mint
description: Auto-pay TempoClaw minting endpoint with Tempo Wallet CLI via MPP, mint $TPC tokens, and return minting status.
---

# TempoClaw ($TPC) MPP Minting

Use this skill when the agent must mint TempoClaw ($TPC) tokens using the Machine Payment Protocol (MPP) via Tempo Wallet CLI.

## Goal

1. Check spendable pathUSD balance first.
2. Execute a paid minting request for $TPCLAW.
3. Handle MPP payment automatically (1 pathUSD = 1,000 $TPCLAW).
4. Verify transaction settlement.
5. Return success status and amount minted.

## Requirements

- Tempo Wallet CLI installed.
- Agent wallet funded with pathUSD for minting payments and fees.
- Network: Tempo Mainnet (Chain ID 4217).

## Spendable Balance Check (required before payment)

Always check spendable balance first. Do not attempt minting if available balance is 0.

Why check spendable balance?
1. **Fee Coverage**: Tempo payments include execution fees paid in pathUSD.
2. **Dust Management**: Small balances (e.g., 0.005 pathUSD) might show in `balanceOf` but are insufficient for the 1 pathUSD mint + overhead.
3. **Settlement Time**: After a top-up, the "spendable" amount may take 20-60 seconds to refresh.

## Task: Make spendable balance available

Run this checklist before minting:

1. Check wallet status and spendable balance.
```bash
tempo wallet status
Top-up wallet if spendable is below the intended mint amount.

Bash
tempo wallet topup --token pathUSD --amount 5
Wait for settlement, then re-verify.

Bash
tempo wallet status
Canonical command (Automated MPP Mint)
Bash
# Replace <AMOUNT> with the desired pathUSD to spend (Max 50).
# 1 pathUSD will result in 1,000 $TPCLAW tokens.
AMOUNT="1"
tempo wallet request https://tempoclaw.xyz/api/mint \
  --method POST \
  --json "{\"amount\":$AMOUNT}"
Raw curl (Manual Credential)
Bash
curl -X POST https://tempoclaw.xyz/api/mint \
  -H "Content-Type: application/json" \
  -H "Authorization: Payment <CREDENTIAL>" \
  -d '{"amount": 1}'
Output contract
If success: Output a JSON summary: {"status": "success", "spent": 1, "received": 1000, "token": "TPCLAW"}.

If fail: Output short error message (e.g., "Insufficient spendable", "Contract Reverted").

If timeout: Check tempo wallet history before retrying to avoid double-minting.

Max Limit: If the request exceeds 50 pathUSD, the agent must stop and report "Exceeds max transaction limit".

Contract Reference
Token Name: TempoClaw

Symbol: TPCLAW

Contract Address: 0xfA4B61D38Ec0ca038283D2fE701E5890ba42C982

Standard: TIP-20 (Tempo Network)
