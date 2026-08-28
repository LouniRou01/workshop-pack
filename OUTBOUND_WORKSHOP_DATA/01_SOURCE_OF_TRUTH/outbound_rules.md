# Outbound rules

These rules define how an agent (or a participant) should move from raw data to a recommended outbound action. They apply on top of the classifications defined in `targeting_brief.md`. See `scenario_registry.csv` in `00_GUIDE/` for worked examples of most of these rules.

## 1. How to select an account and contact

1. Start from `account_tier` in `accounts.csv`. Prefer `tier_1`, then `tier_2`. Do not select `exclude` accounts.
2. Check `02_SOURCE_SYSTEMS/crm_state.csv` before doing anything else. If the account has `do_not_contact: true`, an `open_opportunity: true` with an owner, or an `active_sequence: true`, stop and follow rule 9 instead of proceeding.
3. Within a selected account, choose the contact with the highest `persona_fit` (`priority` over `relevant` over `secondary`). Never select a `not_target` contact.
4. If two contacts have the same `persona_fit`, prefer the one connected to the most recent or strongest signal in `signals.csv`.
5. Confirm the contact's own CRM state in `crm_state.csv` is not already suppressed or actively worked before treating them as available.

## 2. How to combine company and contact fit

Account fit and contact fit are evaluated independently, then combined. Account fit is checked first because it can override contact fit (see the excluded-account example in the targeting brief).

| Account fit | Contact fit | Combined outcome |
|---|---|---|
| strong | priority | Tier 1 |
| strong | relevant | Tier 1 or Tier 2, depending on signal strength |
| strong | secondary | Tier 2 |
| medium | priority | Tier 2 |
| medium | relevant / secondary | Tier 2 or Tier 3 |
| weak | any | Tier 3, unless a strong signal justifies research |
| excluded | any | Do not contact, regardless of contact fit |

A weak account fit is never upgraded by a strong contact fit alone. A strong signal can move a Tier 3 candidate into a research queue, but not directly into Tier 1.

## 3. How to use signal freshness and confidence

Every row in `signals.csv` carries a `signal_strength` and a `verified` flag; treat these as two separate checks:

- **Freshness**: prioritize signals from the last 30 to 60 days. A signal older than 90 days should be treated as weak evidence even if it was originally strong, unless corroborated by a more recent activity or signal.
- **Strength**: `high` strength signals with a clear, specific recommended angle can justify Tier 1 personalization. `medium` strength signals justify a lighter or more exploratory personalization. `low` strength signals are not sufficient on their own and should trigger further research rather than outreach (see `sig_008` and `sig_009` in the current dataset).
- **Verification**: only use `verified: true` signals for personalization. An unverified signal can motivate research but must not appear in outbound copy.
- When two signals conflict, prefer the more recent one, and note the discrepancy in the research brief rather than silently discarding one of them.

## 4. What qualifies as Tier 1, Tier 2, or Tier 3

- **Tier 1**: strong account fit, priority or relevant contact fit, and at least one verified, high- or medium-strength signal from the last 60 days. Action: personalized first-touch message referencing the specific signal, with human review before sending.
- **Tier 2**: medium account fit, or strong account fit with a weaker contact or signal. Action: a semi-personalized message using company-level context (industry, size, geography) rather than a specific event.
- **Tier 3**: weak account fit, or insufficient signal strength/freshness to justify personalization. Action: hold for further research, or route to a lighter-touch nurture flow; do not generate a personalized 1:1 message.

## 5. What evidence can be used for personalization

- Only use facts present in `accounts.csv`, `contacts.csv`, `signals.csv`, or approved research evidence in `03_RESEARCH_EVIDENCE/`.
- A claim used in outbound copy must be traceable to a specific field or `source_url`. If asked "where does this come from," there must be an answer.
- Company description, industry, sub-industry, and headcount are safe to reference directly.
- A signal's `summary` can be referenced directly; its `recommended_angle` is a starting hypothesis, not a verified fact, and should be adapted rather than copied verbatim.

## 6. What the agent must never invent

- Do not invent a pain point, priority, budget, timeline, or quote that is not present in the evidence.
- Do not invent a job title, seniority, or reporting relationship not present in `contacts.csv`.
- Do not invent a company statistic, funding event, or headcount figure not present in `accounts.csv` or a verified signal.
- Do not assume a technology change implies a specific problem; treat technology signals as a reason to ask, not a reason to assert.
- Do not upgrade an account's tier to justify a more personalized message than the evidence supports.

## 7. When to stop because data is insufficient

Stop and hold the record for research (do not draft outreach) when:

- The account has no signal, or only `low` strength / unverified signals.
- The best available contact is `secondary` or `not_target`.
- The account fit is `weak` and there is no corroborating signal.
- Company or contact data is missing fields needed to justify the proposed angle (e.g. no headcount, no clear function).

This corresponds to scenario `scenario_003` (weak signal, weak fit) in the scenario registry: the correct outcome is `hold_for_research`, not a generated message.

## 8. When to route to a human

Always route to a human for approval before any external action, and specifically escalate when:

- The recommended tier is Tier 1 and the message is about to be sent.
- Company and contact fit disagree (e.g. strong account, `not_target` contact).
- A signal is ambiguous, contradictory, or unusually sensitive (executive departure, layoffs, legal issues).
- The account is public sector, nonprofit, or otherwise excluded but a contact still looks individually relevant.
- Any CRM constraint from rule 9 applies.

## 9. When existing CRM ownership, sequences, or suppression flags block outreach

Check `crm_state.csv` for every candidate before generating outreach:

- `do_not_contact: true` at the account or contact level: stop immediately, do not draft a message, and do not propose enrollment in any campaign. This corresponds to scenario `scenario_005`.
- `open_opportunity: true` with a named `owner`: do not create new outreach; instead, notify the existing owner with the new signal or research as context. This corresponds to scenario `scenario_004`.
- `active_sequence: true`: do not enroll the contact in a second, competing sequence. Surface the signal to the owner of the existing sequence instead.
- Missing or `unassigned` owner with no open opportunity and no active sequence: the account is available for a new outbound motion.
