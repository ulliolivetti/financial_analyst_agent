---
name: purchase-costs
description: Record investment purchases using their full acquisition cost and explain the fee-inclusive cost per share. Use when the user asks to add buys, imports broker transactions or screenshots, gives total prices and quantities, or mentions purchase fees, TTF, transaction taxes or average purchase prices.
---

# Purchase costs and effective unit prices

## User's accounting preference

Include all confirmed purchase-related costs in the purchase value: execution
amount, broker fees and additional acquisition taxes such as TTF when applicable.
Then calculate the effective unit acquisition cost:

```text
acquisition_total = supplied_purchase_total
                    + confirmed purchase fees/taxes not already included
effective_unit_cost = acquisition_total / quantity
```

If the user says the supplied total already includes fees, accept it as
fee-inclusive. A separate fee/tax line is not automatically an additional charge:
check whether it is already included in that total before adding it. Never add
the same fee or tax twice.

## Extract and reconcile

1. Identify each instrument, quantity, operation date, quoted execution price,
   total debit and any separate purchase-related fee/tax lines in the source.
2. Use the actual acquisition debit as the basis. A broker's displayed execution
   price can exclude charges and therefore differ from the effective unit cost.
3. Add confirmed extra charges only when they belong to this purchase and are
   excluded from the supplied total. If inclusion or allocation is unclear, ask
   for the broker breakdown or clarification before saving the affected purchase.
4. Do not infer a TTF charge from company size alone, invent its amount, or add an
   estimated tax to a historical transaction as though the broker charged it.
   If applicability must be researched, verify the rules for the instrument and
   transaction date and distinguish an estimate from the actual broker charge.
5. Use exact decimal arithmetic. Keep the source quantity and final total intact;
   round only the displayed unit price. Do not reconstruct the total by multiplying
   quantity by a rounded unit price.

## Record through MCP

- Follow `AGENTS.md` for account/instrument resolution, explicit dates/timezones,
  duplicate checks and uncertain write outcomes.
- Call `record_investment_operation` with `operation_type="buy"`, the actual
  `quantity`, and `value` equal to the **full acquisition total** as a decimal
  string. The tool expects a transaction total, not a unit price.
- Do not record a purchase-related fee or TTF again as an account-level `fee`:
  it already reduces cash and increases the position cost through the buy total.
- If a separate tax is discovered after a buy was saved, explain that a correction
  is needed. Do not insert a duplicate buy or bypass the API; current MCP tools
  do not support editing an existing operation.
- Do not change instruments' market quotes to the calculated purchase unit cost.
  Transaction acquisition cost and current market valuation are different data.

## Explain the result

Show instrument, quantity, fee/tax-inclusive unit cost and acquisition total.
Use `≈` for a rounded unit cost, and identify included extra charges where useful.
After a successful write, include the saved operation ID.

The effective unit cost here applies to this individual purchase. The backend
calculates the overall position's moving-average cost from all relevant operations;
do not overwrite that average with the last purchase price.

## Examples (illustrative, not instructions to write records)

- **Fees already included:** 3 shares, total EUR 175.02 including fees →
  EUR 58.34 per share; send quantity `"3"`, value `"175.02"`. No extra fee entry.
- **Confirmed additional tax:** 10 shares, supplied total EUR 1,005 including
  brokerage, plus EUR 4 TTF explicitly excluded from that total → acquisition
  total EUR 1,009 and unit cost EUR 100.90. Send value `"1009"`, not `"100.90"`.
- **Tax already included:** total debit EUR 1,009 including brokerage and EUR 4
  TTF → keep EUR 1,009, not EUR 1,013.
- **Repeating decimal:** 60 shares, total EUR 200.56 → unit cost
  EUR 3.342666… (display e.g. ≈ EUR 3.342667); retain value `"200.56"` and
  quantity `"60"` rather than using a rounded unit price to recreate the total.
