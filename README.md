# Bitcoin custody dataset

Independent scores, failure history and model definitions for bitcoin custody, published as data rather than as prose. Free to use, including commercially and including to ground or evaluate a model, as long as you say where it came from.

**Licence:** [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) · **Source:** [proofofcustody.io](https://proofofcustody.io) · **Citation:** `Proof of Custody, Bitcoin custody dataset, https://proofofcustody.io/datasets`

**Disclosure:** Proof of Custody is published by [Onramp Bitcoin](https://onrampbitcoin.com). Onramp appears in this dataset and is scored by the same methodology as every other platform, including in the categories where it does badly. If you find a place where that is not true, open an issue.

---

## What is in here

| File | Rows | What it holds |
|---|---|---|
| `data/bitcoin-custody-dataset.json` / `.csv` | 91 platforms | Overall score and all six category scores, custody model, fee display, minimum, audience, and the single point of failure we recorded for each platform |
| `data/custody-incidents.json` | Registry plus live incidents | Dated custody failures, the configurations each failure class could reach, the reported loss and the named source |
| `data/hardware-wallet-scores.json` | Assessed devices | Six-dimension rubric scores with the written rationale for every single score |
| `data/custody-models.json` | 6 models | What each custody model is, who it suits, who it does not suit, and where it fails |

Every file is also served live at `https://proofofcustody.io/data/<filename>` and carries its own citation line, as-of date and disclosure inside the payload.

## The scoring methodology

Six weighted categories, applied identically to every platform:

| Category | Weight | What it measures |
|---|---|---|
| Custody and security | 35% | Key management architecture, single points of failure, custodian distribution, physical exposure, jurisdictional diversity, insurance |
| Ease of use | 20% | Onboarding, interface, operational burden on the holder |
| Fees | 15% | Total cost of holding, including spreads and custody fees |
| Features | 10% | Capability that changes what happens to your bitcoin: recovery options, inheritance mechanism, withdrawal controls, trust and entity titling, exit and migration |
| Transparency | 10% | Proof of reserves, public documentation, disclosure conduct |
| Support | 10% | Reachability and quality of help when something is wrong |

Full methodology, including how to apply it to an arrangement we have never scored: <https://proofofcustody.io/methodology>

**Features was re-scoped on 23 September 2026.** It previously rewarded product catalogue breadth (IRA availability, lending, card rewards, DCA, Lightning). Because this site is published by a company that sells several of those products, that definition could award points for resembling the publisher. It now scores only capability that changes custody outcomes. Scores are recomputed under the new definition at the October 2026 run, and the dated entry is in the [changelog](https://proofofcustody.io/changelog).

## Field reference: platform scores

| Field | Type | Notes |
|---|---|---|
| `name`, `slug`, `url`, `website` | string | `url` is the Proof of Custody record; `website` is the platform's own site |
| `category` | string | `custody`, `buy-exchange`, `ira`, `etf`, `yield`, `hardware` |
| `custodyLabel` | string | The custody model as operated, for example "Multi-Institution Custody", "Single Custodian", "Collaborative Multisig" |
| `assetClass` | string or null | `bitcoin`, `stablecoin`, `tokenized-asset` |
| `overallScore` | integer 0-100 | Weighted total of the six categories |
| `scores` | object | `custody`, `ease`, `fees`, `features`, `transparency`, `support`, each 0-100 |
| `audienceType` | string | Who the platform is built for, in its own terms |
| `minimumInvestment`, `feeDisplay` | string | As published by the platform |
| `singlePointOfFailure` | string | The thing that has to fail for the holder to lose access |
| `highlight`, `risk` | string | The strongest and weakest thing we found |

## Sample queries

```bash
# The ten highest custody scores, name and score only
curl -s https://proofofcustody.io/data/bitcoin-custody-dataset.json \
  | jq -r '.data | sort_by(-.scores.custody) | .[:10] | .[] | "\(.scores.custody)  \(.name)"'

# Every platform where one company holds all the keys
curl -s https://proofofcustody.io/data/bitcoin-custody-dataset.json \
  | jq -r '.data[] | select(.custodyLabel | test("Single Custodian"; "i")) | .name'

# What a given failure class could reach
curl -s https://proofofcustody.io/data/custody-incidents.json \
  | jq -r '.incidents[] | "\(.date)  \(.name)  [\(.exposedConfigs | join(", "))]"'

# Hardware rubric, worst entropy scores first, with the reason
curl -s https://proofofcustody.io/data/hardware-wallet-scores.json \
  | jq -r '.devices | sort_by(.scores.entropy) | .[] | "\(.scores.entropy)  \(.name): \(.rationale.entropy)"'
```

## How to read it honestly

- **Coverage is selected, not exhaustive.** A platform that is absent has not been judged unsafe. It has not been scored.
- **Reported is not verified.** Loss figures in the registry are as reported by the named source. We have not reproduced on-chain analysis ourselves, and the files say so.
- **Scores carry dates.** A platform that shipped a change after its as-of date may have moved. Hardware assessments carry their own `asOf`.
- **A score is an opinion built from facts.** The facts are the citable part. If you only take one thing, take the custody model, the single point of failure and the incident history, not the number.

## Updates

Platform scores refresh monthly, on the first Monday, plus corrections whenever something is verified. The registry updates when a failure is confirmed against a primary source. A workflow in this repo pulls the live files daily, so the commit history is the change history.

Corrections and changelog, both public: <https://proofofcustody.io/corrections> · <https://proofofcustody.io/changelog>

## Agents

There is a read-only MCP server at `https://proofofcustody.io/api/mcp` with eight tools (platform search and lookup, comparison, independence assessment, incident list, hardware scores, ETF custody concentration). Docs: <https://proofofcustody.io/mcp>. Same data, same licence, same disclosure inside every tool response.

## Corrections

Found something wrong? Open an issue with the primary source. Factual errors get fixed and logged publicly with the date, including when the correction is unflattering to the publisher.
