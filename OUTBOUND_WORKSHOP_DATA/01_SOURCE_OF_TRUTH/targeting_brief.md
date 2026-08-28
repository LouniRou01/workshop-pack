# Targeting brief

## Workshop objective

Build a research-backed outbound workflow that selects relevant accounts and contacts, uses evidence to personalize outreach, and routes uncertain cases to a person. This brief defines the targeting model used to classify every account and contact in the dataset, so the labels already present in `accounts.csv` and `contacts.csv` (`icp_fit`, `account_tier`, `persona_fit`) can be reproduced and explained rather than treated as arbitrary.

## Target industries and sub-industries

Strong-fit industries in the dataset:

- **Financial technology**: payments infrastructure, risk and compliance
- **Cybersecurity**: identity security, application security
- **Software**: workflow automation, people operations
- **Data infrastructure**: data quality, developer tooling
- **Energy**: energy management

Medium-fit industries:

- **E-commerce**: marketplace, retail operations
- **Healthcare**: digital health, provider operations
- **Logistics**: last-mile delivery, freight management
- **Manufacturing**: industrial components, factory automation
- **Professional services**: management consulting
- **Energy**: renewable energy (industrial/asset-heavy side)

Weak-fit industries:

- **Education**, **Media**, **Travel**, **Agriculture**, **Design services**, **Legal services** (a professional services sub-industry)

Excluded industries:

- **Public sector** (civic infrastructure)
- **Nonprofit** (social programs)

## Target company sizes

The dataset spans seven employee bands, from `1-10` to `5000+`. There is no single "right" size. Fit depends on the combination of industry and headcount:

- Mid-market bands (`51-200`, `201-500`, `501-1000`) are the most common footprint for strong-fit accounts.
- Very small accounts (`1-10`, `11-50`) are usually weak fit unless the industry is a strong match.
- Very large accounts (`1001-5000`, `5000+`) skew toward public sector and healthcare in this dataset and should be evaluated case by case rather than assumed strong fit.

## Target geographies

The dataset concentrates on Europe, with secondary coverage in North America, Australia, and Singapore:

- **Primary**: France, Germany, United Kingdom, Italy, Spain, Netherlands, Belgium
- **Secondary**: Portugal, Ireland, Switzerland, Austria, Denmark, Sweden, Norway, Finland, Poland, Czechia
- **Opportunistic**: United States, Canada, Australia, Singapore

No geography is excluded outright, but a strong-fit industry in a primary geography should be prioritized over the same industry in an opportunistic geography.

## Priority personas

Contact-level fit (`persona_fit`) uses four values:

| Value | Meaning | Share of contacts |
|---|---|---:|
| `priority` | Directly owns or strongly influences the relevant buying decision | ~41% |
| `relevant` | Connected to the use case, useful for research or multi-threading | ~44% |
| `secondary` | Loosely related, unlikely to be the right first contact | ~6% |
| `not_target` | Not a fit for outbound (e.g. executive assistants) | ~9% |

## Relevant functions and seniority levels

Priority functions, in order of relevance across the dataset:

1. **IT** (CIO, CTO, IT Director, IT Security Manager)
2. **Security** (CISO, IT Security Manager)
3. **Operations** (COO, VP Operations, Director of Operations)
4. **Digital Transformation** (Chief Digital Officer, Head of Digital Transformation)
5. **RevOps, Sales Operations, Marketing Operations**
6. **Innovation** (Head of Innovation, Director of Innovation Programs)
7. **Engineering** and **Product** (VP Engineering, VP Product, Director of Platform)

Seniority levels present in the data, from highest priority to lowest: `executive`, `vp`, `director`, `manager`, `lead`, `individual_contributor`. Executive and VP-level contacts in a priority function should be treated as the strongest starting point for outreach; manager-level contacts in a priority function are still relevant, but usually better as a secondary or multi-thread contact rather than the first touch.

## Excluded sectors

- **Public sector** organizations (civic infrastructure, government tenders): excluded from standard outbound; any relevant signal should be routed for policy and account review rather than direct contact.
- **Nonprofit** organizations: excluded from standard outbound.

These exclusions are encoded as `icp_fit: excluded` and `account_tier: exclude` in `accounts.csv`, and should always resolve to a do-not-contact decision regardless of any other signal.

## Account and contact fit definitions

**Account fit** (`icp_fit` / `account_tier`) is a function of industry fit and, secondarily, company size and geography:

- `strong` / `tier_1`: strong-fit industry, mid-market size, primary or secondary geography
- `medium` / `tier_2`: medium-fit industry, or a strong-fit industry with a less typical size or geography
- `weak` / `tier_3`: weak-fit industry, or very small/very large company with no other strong signal
- `excluded` / `exclude`: public sector or nonprofit, regardless of any other factor

**Contact fit** (`persona_fit`) is a function of function and seniority:

- `priority`: priority function at executive, VP, or director level
- `relevant`: priority function at manager/lead level, or a directly connected function (e.g. Product, Engineering) at any senior level
- `secondary`: tangential function, or priority function at individual-contributor level
- `not_target`: administrative or unrelated roles

## Examples of strong, medium, weak, and excluded accounts

- **Strong**: `acc_001` Northstar Ledger, financial technology, treasury software, 280 employees, France. Contact `con_001` Elena Marin, Chief Information Officer, is `priority`.
- **Medium**: `acc_003` Morrow Mobility, transportation/fleet management, 340 employees, Germany. Contact `con_005` Sofia Keller, Director of Operations, is `relevant`.
- **Weak**: `acc_007` Atelier Nova, design services, 32 employees, Italy. Contact `con_013` Giulia Conti, Studio Director, is `secondary`.
- **Excluded**: `acc_008` Summit Grid Authority, public sector energy infrastructure, 2,200 employees, Spain. Contact `con_015` Lucia Ortega is a `priority` persona by function, but the account exclusion overrides the persona fit, so the account remains do-not-contact.

This last example is deliberate: a strong contact at an excluded account should never override the account-level exclusion. Account fit is checked first.
