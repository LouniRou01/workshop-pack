# Source-system data

These files simulate the operational systems around the canonical accounts and contacts in `01_SOURCE_OF_TRUTH/`. Unlike that folder, records here are expected to be partial, inconsistent, and time-based, the same way a real CRM is.

## `deals.csv`

42 synthetic deals across 37 of the 83 eligible accounts (public sector and nonprofit accounts, marked `account_tier: exclude`, never receive a deal).

- `deal_id`: stable deal identifier.
- `account_id` / `company_name`: joins to `01_SOURCE_OF_TRUTH/accounts.csv`.
- `primary_contact_id`: joins to `01_SOURCE_OF_TRUTH/contacts.csv`; always belongs to the same account.
- `deal_type`: `new_business` or `expansion`.
- `deal_source`: how the deal originated (`outbound`, `inbound`, `partner_referral`, `event`, `expansion`).
- `deal_stage`: one of the six canonical stages (`Initiate` through `Purchasing Decision`).
- `deal_status`: `open`, `closed_won`, or `closed_lost`.
- `amount` / `acv`: annual value in EUR. `tcv`: total contract value over `contract_term_months`.
- `forecast_amount`: `amount` weighted by `probability`.
- `deal_size_band`: bucketed amount, from `<10k` to `250k+`.
- `forecast_category`: `pipeline`, `best_case`, `commit`, or `closed`, derived from stage probability.
- `owner` / `team`: the account executive and their team (`Enterprise`, `Mid-Market`, `SMB`).
- `closed_at` / `closed_lost_reason`: only populated when `deal_status` is `closed_won` or `closed_lost`.

### Distribution

- 46 accounts have no deal at all (available for new outbound).
- 18 accounts have exactly one open deal.
- 9 accounts have a closed-won deal (existing customers).
- 6 accounts have a closed-lost deal (prior attempt, now cold).
- 4 accounts have two deals: an initial closed-won deal plus a later expansion deal.
- `acc_005` (the scenario account already flagged in `crm_state.csv` with an open, owned opportunity) has two open deals, both owned by `alex.morgan`, consistent with its CRM state. This is the account used in scenario `scenario_004` (route to owner).

### Known gap

`crm_state.csv` currently only reflects CRM state for the original 8 scenario accounts and 16 scenario contacts. The `open_opportunity` and `owner` fields for the 37 accounts with a deal in `deals.csv` are not yet reflected there. `crm_state.csv` needs to be regenerated for all 90 accounts and 1,000 contacts, using `deals.csv` as one of its inputs, before the CRM layer is complete.
