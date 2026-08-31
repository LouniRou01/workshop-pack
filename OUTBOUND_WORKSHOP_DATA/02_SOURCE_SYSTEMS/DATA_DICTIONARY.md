# Source-system data

These files simulate the operational systems around the canonical accounts and contacts in `01_SOURCE_OF_TRUTH/`. Unlike that folder, records here are expected to be partial, inconsistent, and time-based, the same way a real CRM is. `crm_state.csv` is the single source of truth for "where does this account/contact stand today." `deals.csv` and `signals.csv` are event/history tables that must stay consistent with it.

## `crm_state.csv`

1,090 rows: one row per account (90) plus one row per contact (1,000).

- `record_type`: `account` or `contact`.
- `record_id` / `account_id` / `company_name`: joins to `01_SOURCE_OF_TRUTH/`.
- `first_name` / `last_name` (contact rows only): denormalized from `01_SOURCE_OF_TRUTH/contacts.csv` for convenience, always blank on account rows, always populated and identical to the source contact on contact rows.
- `customer_status` (account rows only): `prospect`, `pilot`, `customer`, `churned`, or `not_applicable` for excluded accounts.
- `owner`: derived from `deals.csv` when an open deal exists; otherwise `unassigned`.
- `lifecycle_stage`: `lead`, `opportunity`, `customer`, `churned`, or `not_applicable`.
- `open_opportunity` / `active_sequence`: `true` only when the account has at least one `open` deal in `deals.csv`. This is computed from `deals.csv`, not set independently, so the two files cannot drift apart.
- `do_not_contact`: `true` for all 7 excluded (public sector / nonprofit) accounts, and inherited by their contacts.

### Account-level distribution (90 accounts)

| customer_status | Count | Meaning |
|---|---:|---|
| `customer` | 18 (20%) | Active paying customer; has at least one `closed_won` deal |
| `pilot` | 4 | In an active trial; has an open deal |
| `churned` | 6 | Former customer; has a historical `closed_won` deal, lifecycle `churned` |
| `prospect` | 55 | Never signed; may have an open deal, a lost deal, or nothing |
| `not_applicable` | 7 | Excluded (public sector / nonprofit); never has a deal, always `do_not_contact: true` |

Only a subset of contacts per account are marked as actively owned or in an active sequence (1-2 per account with an open deal or customer/pilot status), reflecting that not every contact at an account is being worked.

## `deals.csv`

57 synthetic deals across 43 accounts, generated to match `crm_state.csv` exactly, not independently.

- `deal_id`, `account_id` / `company_name`, `primary_contact_id`: stable identifiers, always consistent with `01_SOURCE_OF_TRUTH/`.
- `deal_type`: `new_business` or `expansion`.
- `deal_stage`: one of the six canonical stages (`Initiate` through `Purchasing Decision`).
- `deal_status`: `open`, `closed_won`, or `closed_lost`.
- `amount` / `acv` / `tcv`: value in EUR. `forecast_amount`: `amount` weighted by stage `probability`.
- `deal_size_band`: bucketed amount, from `<10k` to `250k+`.
- `owner` / `team`: always matches the account's `owner` in `crm_state.csv` when the deal is open.

### Deal status breakdown

- 24 `closed_won`: one per `customer` account (18) and one per `churned` account (6), plus 4 customer accounts have an additional `expansion` deal.
- 27 `open`: 4 pilot accounts, 18 prospect accounts with active pipeline (including `acc_005`, the scenario account used in `scenario_004`), and 4 customer accounts with an open expansion opportunity.
- 6 `closed_lost`: prospect accounts with a prior failed attempt, now cold.
- 46 accounts have no deal at all, and are available for fresh outbound.
- Excluded accounts (public sector / nonprofit) never have a deal.

### Consistency guarantees (validated)

- Every `customer` or `churned` account has at least one `closed_won` deal.
- Every `pilot` account has an open deal.
- `crm_state.open_opportunity` is `true` if and only if the account has at least one `open` deal in `deals.csv`.
- The deal `owner` on any open deal always matches the account `owner` in `crm_state.csv`.
- No excluded account ever appears in `deals.csv`.
