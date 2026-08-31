# Source-system data

These files simulate the operational systems around the canonical accounts and contacts in `01_SOURCE_OF_TRUTH/`. Unlike that folder, records here are expected to be partial, inconsistent, and time-based, the same way a real CRM is. `crm_state_accounts.csv` and `crm_state_contacts.csv` together are the single source of truth for "where does this account/contact stand today." `deals.csv` and `signals.csv` are event/history tables that must stay consistent with them.

## `crm_state_accounts.csv`

90 rows, one per account, joins on `account_id` to `01_SOURCE_OF_TRUTH/accounts.csv`.

- `account_id` / `company_name`: joins to `01_SOURCE_OF_TRUTH/accounts.csv`.
- `customer_status`: `prospect`, `pilot`, `customer`, `churned`, or `not_applicable` for excluded accounts.
- `owner`: derived from `deals.csv` when an open deal exists; otherwise `unassigned`.
- `lifecycle_stage`: `lead`, `opportunity`, `customer`, `churned`, or `not_applicable`.
- `open_opportunity`: `true` if and only if the account has at least one `open` deal in `deals.csv`. Computed from `deals.csv`, not set independently, so the two files cannot drift apart.
- `do_not_contact`: `true` for all 7 excluded (public sector / nonprofit) accounts.

### Account-level distribution (90 accounts)

| customer_status | Count | Meaning |
|---|---:|---|
| `customer` | 18 (20%) | Active paying customer; has at least one `closed_won` deal |
| `pilot` | 4 | In an active trial; has an open deal |
| `churned` | 6 | Former customer; has a historical `closed_won` deal, lifecycle `churned` |
| `prospect` | 55 | Never signed; may have an open deal, a lost deal, or nothing |
| `not_applicable` | 7 | Excluded (public sector / nonprofit); never has a deal, always `do_not_contact: true` |

## `crm_state_contacts.csv`

1,000 rows, one per contact, joins on `contact_id` to `01_SOURCE_OF_TRUTH/contacts.csv` and on `account_id` to `crm_state_accounts.csv`.

- `contact_id` / `account_id` / `company_name`: joins to `01_SOURCE_OF_TRUTH/contacts.csv` and to `crm_state_accounts.csv`.
- `first_name` / `last_name`: denormalized from `01_SOURCE_OF_TRUTH/contacts.csv` for convenience, always identical to the source contact.
- `owner`: the account's `owner` in `crm_state_accounts.csv` if this specific contact is being actively worked, otherwise `unassigned`. Only a subset of contacts per account (1-2 per account with an open deal or customer/pilot status) are marked as actively owned, reflecting that not every contact at an account is being worked.
- `active_sequence`: `true` if this contact is currently in an outbound sequence. Only ever `true` for a subset of contacts at accounts with an open deal.
- `do_not_contact`: currently always inherited from the account's `do_not_contact` in `crm_state_accounts.csv` (no individual opt-outs yet), but the two files are not required to match, e.g. a future update could suppress a single contact without excluding the whole account.
- `last_touch_date`: the date of this contact's most recent activity in `activities.csv`. Derived from `activities.csv`, not set independently, so the two files cannot drift apart. Blank if the contact has never been touched.

There is no `customer_status`, `open_opportunity`, or `lifecycle_stage` column here: those are account-level facts. Join to `crm_state_accounts.csv` on `account_id` to get them for a given contact.

**611 of the 1,000 contacts (61%) have been touched at least once.** This is intentionally decoupled from `owner`/`active_sequence` (only ~88 contacts are *currently* actively owned or in a live sequence): most touched contacts were reached out to at some point and went cold, nobody is actively following up on them today. That gap, touched-but-not-owned, is itself part of the workshop: it's the pool of "warm-but-dormant" contacts a participant might re-engage using a new signal.

## `deals.csv`

57 synthetic deals across 43 accounts, generated to match `crm_state_accounts.csv` exactly, not independently.

- `deal_id`, `account_id` / `company_name`, `primary_contact_id`: stable identifiers, always consistent with `01_SOURCE_OF_TRUTH/`.
- `deal_type`: `new_business` or `expansion`.
- `deal_stage`: one of the six canonical stages (`Initiate` through `Purchasing Decision`).
- `deal_status`: `open`, `closed_won`, or `closed_lost`.
- `amount` / `acv` / `tcv`: value in EUR. `forecast_amount`: `amount` weighted by stage `probability`.
- `deal_size_band`: bucketed amount, from `<10k` to `250k+`.
- `owner` / `team`: always matches the account's `owner` in `crm_state_accounts.csv` when the deal is open.

### Deal status breakdown

- 24 `closed_won`: one per `customer` account (18) and one per `churned` account (6), plus 4 customer accounts have an additional `expansion` deal.
- 27 `open`: 4 pilot accounts, 18 prospect accounts with active pipeline (including `acc_005`, the scenario account used in `scenario_004`), and 4 customer accounts with an open expansion opportunity.
- 6 `closed_lost`: prospect accounts with a prior failed attempt, now cold.
- 46 accounts have no deal at all, and are available for fresh outbound.
- Excluded accounts (public sector / nonprofit) never have a deal.

### Consistency guarantees (validated)

- Every `customer` or `churned` account has at least one `closed_won` deal.
- Every `pilot` account has an open deal.
- `crm_state_accounts.open_opportunity` is `true` if and only if the account has at least one `open` deal in `deals.csv`.
- The deal `owner` on any open deal always matches the account `owner` in `crm_state_accounts.csv`.
- No excluded account ever appears in `deals.csv`.

## `activities.csv`

1,068 rows: the interaction log behind `crm_state_contacts.last_touch_date`. Every touched contact has at least one row here; every row belongs to a non-suppressed (`do_not_contact: false`) contact.

- `activity_id`, `account_id`, `contact_id`, `company_name`: identifiers, joins to `01_SOURCE_OF_TRUTH/`.
- `activity_type`: `email_sent`, `email_opened`, `email_replied`, `linkedin_connection_sent`, `call_placed`, `call_connected`, `voicemail_left`, `meeting_scheduled`, `meeting_held`.
- `channel`: `email`, `linkedin`, `phone`, or `video`.
- `direction`: `outbound` (rep/sequence-initiated) or `inbound` (`email_replied`, `call_connected`).
- `activity_date`: always on or after the contact's `contact_created_at` in `01_SOURCE_OF_TRUTH/contacts.csv`, and never after 2026-08-28 (dataset's "today").
- `owner`: who performed the touch. For contacts still `unassigned` in `crm_state_contacts.csv` today (touched historically but nobody currently owns follow-up), this is the rep who made the original outreach, not the current owner.
- `related_deal_id`: set when this contact is a deal's `primary_contact_id` in `deals.csv`.
- `related_signal_id`: set on the first touch for the 10 contacts referenced in `signals.csv`, i.e. the outreach that was triggered by that signal.
- `related_campaign_id`: reserved for when `campaigns.csv` is populated; blank for now.
- `outcome`: `sent`, `no_response`, `opened`, `replied_neutral`, `replied_positive`, `booked`, `held`, `connected`.

### How a contact's touch history is built

Each touched contact gets one of several activity "arcs", 1 to 5 rows forming a coherent story rather than independent random rows:

| Arc | Rows | Used for |
|---|---|---|
| `single_cold` / `double_cold` | 1-2 | Most touched-but-unowned contacts: a cold email (or two), no response. |
| `opened_stalled` | 2 | Opened the email, never replied. |
| `replied_stalled` | 3 | Replied, but nobody followed up, an intentional "dropped ball" for realism. |
| `warming_multi` | 3 | LinkedIn touch + email, some engagement, not yet owned. |
| `engaged_meeting` | 5 | Currently actively owned contacts: full email → open → reply → meeting arc. |
| `call_engaged` | 5 | Currently actively owned contacts: phone-led arc ending in a booked meeting. |

`engaged_meeting`/`call_engaged` are only used for the ~88 contacts with a real `owner` in `crm_state_contacts.csv`; everyone else gets a colder arc.

### Consistency guarantees (validated)

- `crm_state_contacts.last_touch_date` equals the max `activity_date` for that contact, exactly, for all 611 touched contacts.
- Zero activities for any `do_not_contact: true` contact.
- Every `active_sequence: true` contact has at least one activity.
- No `activity_date` before the contact's `contact_created_at`, and none after 2026-08-28.
- Every `related_deal_id` and `related_signal_id` resolves to a real row in `deals.csv` / `signals.csv`.

## `emails.csv`

794 rows: the actual email content behind the `email_sent`, `email_replied`, and `meeting_scheduled` (email-channel) rows in `activities.csv`. `email_opened` activities don't get their own row here, they're an engagement event on a message that's already logged.

- `email_id`, `activity_id`: joins 1:1 to a row in `activities.csv`. Every email-channel `email_sent`/`email_replied`/`meeting_scheduled` activity has exactly one row here.
- `thread_id`: `thr_<contact_id>`, groups every message with a given contact into one discussion, in chronological order.
- `account_id`, `company_name`, `contact_id`: joins to `01_SOURCE_OF_TRUTH/`.
- `direction`: `outbound` (from the rep) or `inbound` (from the contact).
- `message_type`: `outbound_send`, `inbound_reply`, or `meeting_confirmation`.
- `from_email` / `to_email`: the rep's address is `<owner>@lumenops.example` (the fictional vendor selling org); the contact's address comes from `01_SOURCE_OF_TRUTH/contacts.csv`.
- `subject` / `body`: the actual message text.
- `related_deal_id` / `related_signal_id`: same meaning as in `activities.csv`, only set when the message content actually reflects that deal/signal.
- `template_id`: which content pattern generated this message, see below.

### How the content was generated

Not every message is hand-written, most cold outreach reuses a small set of templates, personalized with real fields (company name, job title, function, industry), the same way real SDR sequences work:

| `template_id` | Count | What it is |
|---|---:|---|
| `tmpl_01` .. `tmpl_10` | ~720 | Cold outbound templates, picked deterministically per contact, filled with their real company/job_title/function/industry. |
| `signal_grounded` | 7 | Fully custom first-touch email that references the actual signal in `signals.csv` (only for the 7 signals with a genuine, usable `recommended_angle`; the 3 "weak signal" / "no signal" / "do not contact" signals intentionally fall back to a generic cold template instead of leaking their internal meta-guidance text into a sent email). |
| `follow_up_bump` | ~85 | The second email in a `double_cold` activity arc. |
| `reply_positive` / `reply_neutral` / `reply_negative` | ~58 | Inbound reply content, matching the `outcome` on the corresponding `activities.csv` row. |
| `meeting_confirmation` | ~15 | Short outbound logistics email following a booked meeting in the `engaged_meeting` arc. |

### Consistency guarantees (validated)

- 1:1 coverage: every email-channel `email_sent`/`email_replied`/`meeting_scheduled` row in `activities.csv` has exactly one row here, and vice versa.
- `sent_date` always matches the linked activity's `activity_date`.
- Zero rows for `do_not_contact: true` contacts.
- `from_email`/`to_email` always resolve correctly to the rep's `@lumenops.example` address and the contact's real address in `contacts.csv`.
- Every `related_deal_id`/`related_signal_id` resolves to a real row, and is only set when the content actually reflects it.
- Messages within a thread are in chronological order.

