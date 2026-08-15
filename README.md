# Hydro-Sentinel: Judge Audit and Redesign
### Forensic review against the Mireye x Delhi University Build Brief, prepared for Team Neer-eye (Avishi Agarwal, Mrudduni J Modha, Vidit Arora)

This reads Team_Neer-eye.md and the implementation plan strictly against mireye-du-build-brief.md, the actual rules you are being judged on. It is not a review of whether the project is good in general. It is a review of whether it is *this* project.

---

## Headline finding, before the 20 parts

Two things are doing almost all of the damage, and they compound each other.

**First, the output is a per-parcel buy or don't-buy risk memo, and the brief names that exact shape and excludes it.** "Not site selection. That is our own product." Mireye's own site shows "Property Diligence" as a named vertical, which you were shown directly earlier. A High Risk / Moderate Risk / Low Risk memo for a specific address, meant to be attached to a land transaction file, is Property Diligence with different branding. A judge does not need to read your rubric to reject this. They need to read one sentence of your output format.

**Second, as written, the agent is not an agent by the brief's own definition.** The implementation plan states the planner is "deterministic, not LLM," the evidence check is "rule-based, not LLM," and the escalation is "hardcoded, not left to the model to decide." The brief defines an agent as software that "makes a decision you did not hardcode." By the plan's own description, the only non-hardcoded component is the LLM writing memo prose at the end. That is API calls, then rules, then a score, then an LLM writes it up. The brief explicitly distinguishes that shape from an agent.

Everything below explains why, and exactly how to fix both without discarding the genuinely strong parts, which are your domain research and your citation discipline.

---

## Part 1: Requirement-by-requirement audit

GREEN = satisfies. YELLOW = partial or ambiguous. RED = materially misses. BLACK = likely fatal on its own.

| # | Requirement | Brief demands | What Hydro-Sentinel does now | Status | Fix |
|---|---|---|---|---|---|
| 1 | Agent at the centre | A decision you did not hardcode | Planner, evidence check, and escalation are explicitly deterministic; only memo prose is LLM-generated | BLACK | Move sufficiency judgment and ranking into real agent reasoning, Part 7 |
| 2 | Not site selection | Must not be "is this parcel good" | Output is a per-parcel buy-risk tier, matching Mireye's own Property Diligence vertical | BLACK | Reframe to an institutional ranking decision, Part 6 |
| 3 | The join is the interesting part | Fuse with a dataset nobody has combined with Mireye | NID hazard class and USGS overdraft status are already-published risk labels; Mireye's terrain and soil are not concretely computed anywhere in the current design | RED | Add one unconventional dataset and make Mireye mathematically load-bearing, Parts 9 to 10 |
| 4 | Enrichment beyond any single field | A signal no source states on its own | Current tiers are close to "read NID's label, read USGS's label, put both in a memo" | RED | Build the compound formula in Part 9 |
| 5 | A specific question | One sentence, testable | "Assess water risk for any US parcel" is a topic | YELLOW | Adopt the Part 6.1 statement |
| 6 | Second dataset preferably unconventional | "If the join is obvious, pick again" | NID and USGS are the two most obvious sources in this exact domain | RED | Add RMA Summary of Business, Part 10 |
| 7 | No arbitrary weighted scores | Physically or statistically justified | Weights partly trace to NID's own categories, partly invented to combine two tiers into one | YELLOW | Redesign in Part 8 |
| 8 | Decisions traceable to source fields | Every claim cited | Dam ID, Station ID, and timestamp citation is genuinely strong | GREEN | Keep as-is |
| 9 | Tested on 10 to 20 known cases, honest reporting | A concrete evaluation set | Not designed yet | RED | Part 14; ground truth exists via FEMA's HHPD grant awards and ASDSO records |
| 10 | State limitations, resolution, vintage, blind spots | Explicit, not buried | Present in places (staleness, restricted EAPs) but scattered, not a dedicated section | YELLOW | Fold into the short write-up's limits section |
| 11 | A few pages, not a thesis | Question, datasets, enrichment, evaluation, limits | Team_Neer-eye.md runs to business-plan length: finance, partnerships, legal, future roadmap | RED | Cut per Part 17, keep the rest as an internal doc |
| 12 | Budget-aware design | Design around the 300-credit parcel fields | Not addressed with numbers anywhere in the current plan | YELLOW | Part 15; redesign needs zero parcel-level fields |
| 13 | A 2 to 3 minute demo of a decision | Visible reasoning, not an architecture explainer | No demo script exists | RED | Part 13 |

Two BLACKs on requirements this explicit is not a "polish it" situation. It is a "the core shape needs to change" situation. That is what the rest of this document does.

---

## Part 2: The 10 most likely rejection reasons, ranked

1. **"This is Property Diligence with different words."** What a judge saw: a per-parcel High/Moderate/Low memo. Idea-level. Fatal. Fix: Part 6.
2. **"Remove Mireye and this still works."** NID gives the hazard label, USGS gives the trend, both usable without Mireye; terrain and soil read as decoration, not load-bearing input. Dataset-level and architecture-level. Fatal. Fix: Part 9.
3. **"Where is the agent? I see a pipeline."** Planner, checks, and escalation are all described as hardcoded in your own implementation plan. Agent-level. Fatal. Fix: Part 7.
4. **"NID and USGS are the two most obvious datasets for this exact problem."** Anyone building a dam-and-water project reaches for these first. Dataset-level. Major. Fix: Part 10.
5. **"This reads like a startup pitch deck, not a technical submission."** Finance, legal, partnerships, and a five-year roadmap outweigh the agent loop and evaluation in the actual write-up. Communication-level. Moderate. Fix: Part 17.
6. **"No evidence this was tested against anything real."** No stated 10 to 20 case run, no honest failure count anywhere in the documents. Evaluation-level. Major, but fast to fix. Fix: Part 14.
7. **"If there is a demo, it is probably a PDF, and the brief warns against exactly that."** The deliverable centers on a memo document. Demo-level. Major. Fix: Part 13.
8. **"Acute and chronic read as two projects sharing a cover page."** Separate retrieval, separate rubrics, separate scoring, stapled into one memo. Product-level. Moderate. Fix: Part 5.
9. **"The rubric has the shape of arbitrary weights."** YAML weights combine two already-existing classifications into one tier without a stated physical justification for the combination itself. Architecture-level. Moderate. Fix: Part 8.
10. **"I cannot tell who uses this or what they do next."** "A consultant attaches this to a file" is a vague, indirect action, not a decision. Product-level. Moderate. Fix: Part 11.

One concrete, non-strategic issue worth fixing regardless of anything else: Team_Neer-eye.md has two sections both titled "4.1 / 4.3 Economic Value & Cost Savings" with "Technical feasibility" sandwiched between them as 4.2. That reads as a copy-paste seam in the document. Small, but a judge notices document hygiene, and it costs nothing to fix.

---

## Part 3: Is the current enrichment actually an enrichment?

Working through the brief's own diagnostic questions honestly:

- **Is dam risk already encoded by NID?** Yes. Hazard Potential Classification and Condition Assessment are exactly that, published, government-issued risk labels.
- **Is aquifer risk already encoded by USGS?** Yes. Depth-to-water trend and overdraft status are the risk signal, already computed by USGS.
- **Is the scoring a weighted combination of existing risk indicators?** Largely yes, as currently designed.
- **Does Mireye materially change the conclusion?** Not concretely, as written. Terrain and soil are mentioned as informing geotechnical risk in general language, but nothing in the plan computes a number from them that feeds the tier.
- **Could the same result be produced without Mireye?** Mostly yes. A consultant with NID and USGS access alone gets close to the same two labels.
- **Is the current output data aggregation plus rubric plus LLM memo generation?** That is an accurate description of the current design.
- **What derived quantity does no source state?** None, currently. "Stranded Asset" is a label applied to a combination of two other labels, not a newly computed physical or economic quantity.

**Verdict: this is primarily data aggregation, and it is weak against the brief as currently scoped.**

This is not a verdict on the domain, which is genuinely strong, well-researched, and underserved. It is a verdict on where the computation currently happens. The fix is not a new idea. It is making Mireye's fields do arithmetic that NID and USGS cannot do on their own, which Part 9 defines exactly.

---

## Part 4: Candidate enrichments

Full specification for the strongest candidate, then a ranked, condensed comparison of the rest.

### Candidate 1 (recommended): Compound Dam-Rehabilitation Priority

**A. User question.** Given a state's list of high-hazard, poor-condition dams, which ones should move up the inspection and rehabilitation queue because the land and structures below them are also economically water-stressed, making both the exposure and the recovery cost worse than the dam's hazard label alone suggests?

**B. Second dataset.** USDA RMA State/County/Crop Summary of Business, filtered to drought-related cause-of-loss indemnity by county and year. Confirmed real, public, downloadable as pipe-delimited flat files, no key required, going back decades and current through 2026.

**C. Publisher.** USDA Risk Management Agency.

**D. Join key.** County FIPS code (RMA is county-level) joined to dam location by spatial containment (which county the dam and its downstream buffer sit in), and to USGS station by nearest-station-to-county-centroid or documented aquifer region.

**E. Mireye fields required.** Elevation and slope near each dam (confirmed categories: terrain), building or structure counts within a downstream buffer (confirmed category: built environment), land cover or crop type if available (confirmed category: land cover). Exact field names must be pulled fresh from `GET /v1/meta/fields`, since only `elevation`, `slope_degrees`, and analogues to `housing_units_within_1km` are confirmed by name in the brief's own worked example.

**F. Second-dataset fields required.** NID: hazard potential classification, condition assessment, dam location, crest elevation, EAP-on-file status. USGS: depth-to-water trend, overdraft status, station ID. RMA: county, crop, cause of loss (drought), indemnity per insured acre, year.

**G. Derived enrichment.** Neither NID, USGS, Mireye, nor RMA states this directly. NID does not know what is downstream or what the local economy looks like. USGS does not know where the dams are. RMA does not know about dams or elevation. Mireye does not classify hazard or economic loss. We derive: expected downstream exposure (from Mireye structures and elevation, weighted by NID's own hazard and condition categories), then multiply by a water-stress multiplier built from USGS trend status and RMA's trailing drought-indemnity trend for that county. See Part 9 for the exact formula.

**H. Why it matters.** Two dams with an identical NID "High Hazard, Poor Condition" label get different real-world consequences depending on what sits below them and whether that land is already an agricultural insurance loss magnet. State dam-safety budgets are finite. This tells a real allocator which identical-looking labels actually deserve to be unequal in priority.

**I. Agent decision.** Ranks all dams in the case universe, flags the top tier as "escalate for inspection review," and explicitly marks any dam where evidence was too thin to rank confidently as "insufficient evidence, do not rank" rather than guessing.

**J. Ground-truth possibility.** Yes. FEMA's Rehabilitation of High Hazard Potential Dams grant program publishes award information for fiscal years 2019 to 2024, and state dam-safety emergency actions are generally public record. Use this to retrospectively check whether dams that already received attention would have ranked highly using only data that predates that attention. This is a plausibility check, not a causal claim, and should be reported as exactly that.

**K. Mireye dependency.** No, it would not work without Mireye at the same quality, because the structure count and elevation-based exposure estimate is the only part of the pipeline that is not already published by a federal risk agency. Removing Mireye removes the actual computation, not just a decoration.

**L. Demo strength.** High. A ranked list where dam A visibly outranks dam B despite an identical NID label, with the reason stated in plain language and traced to specific fields, is exactly the shape of the brief's own tower-site example.

**M. Implementation complexity.** Moderate. The hardest part is the naive downstream-exposure estimate from elevation and dam crest height, which should be built and labeled explicitly as a screening-grade approximation, not a hydraulic flood model.

**N. Estimated Mireye credit cost.** Roughly 6 to 10 ordinary fields per dam location at 1 credit each. Even 50 dam locations across 2 to 3 pilot states costs under 500 credits, a small fraction of the 25,000 monthly budget per person.

### Other candidates, ranked and condensed

| Rank | Candidate | Second dataset | Derived signal | Agent decision | Verdict |
|---|---|---|---|---|---|
| 2 | Grid-outage-adjacent dam risk | EIA infrastructure data, paired with Mireye's confirmed `nearest_transmission_line_distance_m` field | Dams whose failure would also plausibly disable power to the substation serving the evacuation zone | Flag "compound emergency," not just "dam emergency" | Strong, genuinely novel, works well as an additive signal inside Candidate 1 rather than a separate project |
| 3 | Insurance non-renewal correlate | State FAIR Plan or Citizens policy-count-by-county filings (California, Florida both publish this) | Where the private insurance market has already priced in a hazard the public label has not caught up to | Flag "market already knows, public data lagging" | Strong idea, but it pivots away from dam and aquifer entirely; treat as a separate future project, not a patch on this one |
| 4 | EAP transparency public-interest mapper | Census population near dams (confirmed Mireye category, but a conventional dataset) | Which dams have no public Emergency Action Plan despite dense downstream population | Public alert, not a ranking | Simpler and safer on the site-selection axis since the user is the public, not a buyer, but weaker on "unconventional dataset" |
| 5 | Irrigation stranded-asset index | Same RMA data, aquifer-only | Whether cropland's insurance-claim trend is already showing the economic effect of aquifer decline | Flag counties for ag-lender attention | Good fallback if the team wants a single branch instead of the compound reframe; weaker on novelty since it is closer to the original chronic branch |
| 6 | Farm sale-price divergence | County assessor or USDA NASS land value records | Where sale prices have not yet adjusted for known aquifer decline | Flag "market has not priced this in" | Interesting, but it drifts straight back toward "should I buy this land," reintroducing the site-selection problem; deprioritize |
| 7 | Wildfire route and dam-inundation shared chokepoint | NIFC or USFS fire perimeter data | Roads that are both the sole wildfire evacuation route and inside a dam's inundation shadow | Flag "single point of failure for two different disasters" | Highest novelty, but highest complexity and hardest ground truth to obtain in the time available; good future-work note, not a hackathon build |

---

## Part 5: Should the two-branch design survive?

Considering the questions directly: yes, it currently makes the project look like two loosely related tools, it does dilute the core question, it does make the enrichment less clear because a reader cannot tell which branch actually contains the novel computation, and it does make Mireye's contribution look smaller because each branch only uses a slice of it.

**Verdict: Option D. Reframe both under a single derived decision problem.**

Not Option C, because the underlying insight that a dam failure is worse when the land below it is already water-stressed is genuinely good and is the whole reason a compound signal exists at all. Not Option A, because "acute" and "chronic" as separate named branches with separate rubrics is exactly what reads as two projects. Under Candidate 1, aquifer and dam data stop being two outputs and become two inputs to one number, which is a smaller change to your existing work than it sounds like: you keep both APIs, both datasets, and almost all of your domain research. You delete the idea that they produce two separate scores.

---

## Part 6: The redesigned project

### 6.1 Problem statement

Which high-hazard, poor-condition dams should move up a state's inspection and rehabilitation queue because the land and structures below them are also water-stressed, making a failure both more damaging and slower to recover from than the dam's hazard label alone would suggest?

### 6.2 Product definition

A ranking tool for state dam-safety offices and grant administrators that reorders their own high-hazard dam list using a compound exposure signal Mireye, NID, USGS, and USDA crop-insurance data produce only when joined together.

### 6.3 Agent definition

Given a region, the agent pulls Mireye structure and terrain data plus NID and USGS and RMA records for every high-hazard dam in scope, derives a compound exposure score no single source states, ranks the dams, and recommends which ones deserve escalated review, stating exactly which fields drove each ranking.

### 6.4 The exact agent loop, with what is genuinely dynamic marked

| Step | What happens | Dynamic (agentic) or fixed (deterministic) |
|---|---|---|
| 1 | User asks a region-scoped question | Input |
| 2 | Agent interprets the question and the implicit intent (rank vs. lookup vs. compare two named dams) | Dynamic, genuine LLM interpretation, phrasing varies |
| 3 | Agent selects the base Mireye field set | Fixed, for cost predictability |
| 4 | Agent calls Mireye for each dam location | Deterministic tool call |
| 5 | Agent calls NID, USGS, and RMA | Deterministic tool call |
| 6 | Agent computes the exposure and compound-stress formula | Deterministic math, formula fixed in advance |
| 7 | Agent evaluates whether evidence is sufficient to trust the score | **Dynamic.** This is the step that must not be hardcoded. Borderline cases (partial data, an old NID condition rating, a station far from the dam) require a judgment call, not a fixed threshold |
| 8 | Agent decides whether to widen the search radius, try the next-nearest station, or abstain | Dynamic, bounded to 1 to 2 retries |
| 9 | Agent ranks the case universe and assigns escalate, routine, or insufficient-evidence flags | Dynamic, since identical raw scores can get different flags depending on confidence |
| 10 | Agent produces the ranked, action-oriented output | Deterministic formatting of a dynamic decision |
| 11 | Agent exposes every field, source, and timestamp behind each rank | Deterministic |

Step 7 through 9 is the part that actually earns the word "agent." Everything else in this table could be written by a strong junior engineer in an afternoon. That block is the part worth spending real design time on, more than the UI, more than the exact dataset choice.

---

## Part 7: Making the agent actually agentic

### Tools the agent can call

- Mireye field discovery (`/v1/meta/fields`)
- Mireye quote (`/v1/fetch/quote`)
- Mireye fetch, batched (`/v1/fetch/batch`)
- NID lookup by radius
- USGS station lookup and trend calculator
- RMA county indemnity lookup
- Geocoder, for region-name to coordinate resolution
- Naive downstream-exposure calculator (a function, not a tool call, see Part 9)
- Evidence sufficiency evaluator
- Confidence tagger

### Decisions that must not be hardcoded

- Whether a given dam's evidence is sufficient to rank with confidence
- Whether to escalate (widen radius, try another station) or abstain
- How to flag two dams with the same raw score but different data confidence
- Whether two dams are even comparable given how different their evidence quality is

### Deterministic functions, and this must stay honest

- The exposure formula itself (Part 9)
- Distance and radius geometry
- The severity lookup table drawn from NID's own categories
- Percentile calculations against the RMA distribution

Do not describe the deterministic functions as agentic in the write-up. The brief specifically warns against this, and judges who have read a few hundred submissions will recognize the pattern immediately.

---

## Part 8: Fixing the arbitrary-score problem

| Element | Defensible? | Why |
|---|---|---|
| NID hazard potential classification as an input | Yes | It is the regulator's own category, not your invention |
| NID condition assessment as an input | Yes | Same reasoning |
| A severity lookup table combining hazard and condition | Yes, if built as a small, published, interpretable table rather than a continuous weighted score | Each cell has a plain-language justification you can defend in one sentence |
| USGS overdraft or critical-decline status | Yes | Official designation in states that have one |
| A generic 1 to 10 water-stress score | No, as currently implied | This is exactly the arbitrary weighting the brief warns against |
| Percentile-based RMA multiplier (top quartile, trailing 5-year growth) | Yes | Statistically grounded against the actual distribution of counties, not a number picked because it felt right |
| A single blended acute-plus-chronic number | No | Conflates two different kinds of physical risk into one uninterpretable digit |

Replace any remaining continuous weighted average with additive, inspectable increments (Part 9's multiplier design), each independently justified and independently removable, and keep the two physical inputs (structural exposure, water stress) visible as two numbers even inside one ranking, never collapsed into a single unexplained score.

---

## Part 9: The enrichment, defined mathematically

**Inputs, Mireye:**
- M1: building or structure count within a downstream buffer around dam i
- M2: elevation and slope near dam i, used to build a naive "below crest elevation" mask
- M3: land cover or crop type, if resolvable, for the surrounding parcels

**Inputs, second sources:**
- D1: NID hazard potential classification for dam i
- D2: NID condition assessment for dam i
- D3: USGS depth-to-water trend and overdraft status for the nearest station
- D4: RMA drought-cause indemnity per insured acre, county-level, trailing 5 years

**Step one, exposure:**

E_exposure(i) = M1_downstream_count(i) x severity(D1, D2)

where severity is a small published lookup table (for example, High hazard with Poor or Unsatisfactory condition = 1.0, High with Fair = 0.6, Significant with Poor = 0.5), grounded directly in NID's own official categories rather than invented.

**Step two, compound water stress multiplier:**

multiplier(county) = 1.0, plus 0.25 if D3 shows the county in documented critical or overdraft decline, plus 0.25 if D4 is in the top quartile nationally or has grown over the trailing 5 years

**Step three, the derived signal:**

E_compound(i) = E_exposure(i) x multiplier(county containing i)

**What E_compound physically represents:** the expected severity of a dam failure at location i, adjusted upward when the surrounding agricultural economy is already water-stressed, since land that floods and then has no reliable water table left afterward does not recover the way healthy land does.

**Why no source states this:** confirmed in Part 3 and Part 4.G. Each source contributes a fact none of the others have.

**How it becomes a decision:** dams are ranked by E_compound within the case universe, and dams above a percentile threshold in that specific universe (not a fixed global number) are flagged for escalation.

**Uncertainty handling:** every E_compound carries a confidence tag (full data, partial data, widened radius, or insufficient) rather than being presented as one clean number regardless of how complete the underlying evidence was.

**Validation:** compare the ranking against FEMA HHPD grant award recipients and documented state dam-safety emergency actions for the same pilot states, as a retrospective plausibility check.

---

## Part 10: Second-dataset strategy

NID and USGS are both credible and both too obvious for this specific brief, which explicitly rewards an unconventional join. The fix is not to replace them, since they are the correct authoritative sources for hazard and groundwater trend. The fix is to add a third, genuinely unconventional dataset that does real computational work.

**Recommended: USDA RMA State/County/Crop Summary of Business**, filtered to drought-related cause-of-loss. Confirmed real and public in this session (pipe-delimited flat files, county level, current through 2026, no key required). It creates a genuinely new signal when fused with Mireye and NID because it is the only one of the three that tells you whether the physical risk has already turned into realized economic loss, which neither a hazard classification nor a groundwater trend can say on its own.

Secondary option worth keeping in your back pocket: EIA infrastructure data paired with Mireye's confirmed `nearest_transmission_line_distance_m` field, layered on top of the RMA signal rather than replacing it, to flag dams whose failure would also plausibly cut power to the response effort.

---

## Part 11: Challenging the product definition

- **Exact user:** a state dam-safety office or FEMA HHPD grant reviewer with a long list of high-hazard dams and a finite inspection or rehabilitation budget, not a land buyer.
- **Decision they make:** which dams to prioritize for the next inspection or grant application cycle.
- **Action after the output:** the ranked list feeds directly into a grant application or an internal budget request, a concrete next step, not "attach to a file."
- **Why this instead of GIS plus government portals plus an analyst:** the analyst already has NID and USGS. What they do not have is a fast way to check every dam on their list against county-level economic water stress, which currently means manually cross-referencing spreadsheets.
- **What the agent automates:** the cross-referencing and ranking, not the underlying facts, which were always public.
- **The real product:** the ranking and its stated reasoning, not the memo format. Drop the "Risk Disclosure Memo" framing entirely, since that framing is what pulls the whole project back toward Property Diligence.

Redesigned surface: **user input -> agent -> ranked decision -> evidence -> action**, not **user input -> report**.

---

## Part 12: Minimum viable demo UI

1. **What question is being answered:** shown as a literal sentence at the top of the screen, not implied.
2. **What did the agent decide:** a ranked list, top to bottom, no scrolling required to see the top 5.
3. **What data did it retrieve:** a visible, live log of tool calls as they fire, not a static "data sources" footer.
4. **What enrichment did it calculate:** the two-part breakdown (exposure, then multiplier) shown per dam, not hidden behind a single score.
5. **Why did it decide that:** click into any ranked dam, see the exact fields, sources, and timestamps behind its number.
6. **What action should the user take:** an explicit "escalate," "routine," or "insufficient evidence, do not rank" label per dam.
7. **Can I verify the evidence:** every number links back to its source ID.

Do not spend design time on visual polish beyond this. A plain, readable table that does all seven of the above beats a beautiful dashboard that hides the reasoning.

---

## Part 13: Demo script, 2 minutes 30 seconds maximum

- **0:00 to 0:15** — State the question on screen exactly as written in Part 6.1.
- **0:15 to 0:35** — Select a real pilot region, show the agent parsing intent and deciding which dams are in scope.
- **0:35 to 1:10** — Show the tool calls actually firing (Mireye, NID, USGS, RMA) in a live log, then the two-part enrichment number appearing per dam.
- **1:10 to 1:40** — Show the ranked list, click into the top-ranked dam, show why it outranks a dam with an identical NID label, tracing every claim to its source.
- **1:40 to 2:05** — Show one live abstention: a dam where evidence was too thin, and the agent says so instead of guessing. This single moment does more to prove "not hardcoded" than anything else in the demo.
- **2:05 to 2:30** — State the 10 to 20 case evaluation result out loud, including the honest failure count, then stop talking.

---

## Part 14: Evaluation design

- **Positive case:** a documented high-hazard, poor-condition dam that later received FEMA HHPD grant funding or a state emergency intervention. The agent should have ranked it highly using only data available before that funding or intervention.
- **Negative case:** a high-hazard dam rated satisfactory or recently rehabilitated, with no emergency funding history and no downstream agricultural stress. The agent should rank it lower.
- **Selection, to avoid cherry-picking:** pull the complete list of high-hazard, poor or unsatisfactory condition dams for 2 to 3 pilot states from NID directly, which per ASDSO's own figures should yield a natural sample in the 15 to 25 dam range per state, and test on all of them rather than hand-picking dramatic stories.
- **Ground truth source:** FEMA's HHPD grant award pages (fiscal years 2019 to 2024 are published) and documented state dam-safety emergency actions. Verify before building the evaluation set whether individual dams are named at the award or project level, which is likely but should be confirmed rather than assumed, since this needs a short verification pass, not a guess.
- **Metrics:** whether the agent's top-ranked tier includes a meaningful share of the dams that actually received funding or intervention, reported honestly as a count, for example 12 of 20, not as a polished percentage that implies more precision than the sample supports.
- **Expected failure modes to report, not hide:** NID condition ratings are not updated every year, many dams have no public inundation map or EAP, RMA data is county-level so it cannot distinguish which specific farms near a given dam are exposed, and the sample size at 15 to 25 dams per state limits how much confidence any single number deserves.
- **Uncertainty reporting:** tag every case's confidence based on data completeness, and show the low-confidence cases separately from the high-confidence ones rather than averaging them together.

Do not invent a performance number before this is actually run. If the honest result is 11 of 20, say 11 of 20.

---

## Part 15: Mireye credit budget

- Lock the exact field list and current pricing with `GET /v1/meta/fields` before writing retrieval code, since only category names, not exact field strings beyond the brief's own worked example, are confirmed right now.
- Quote every batch with `/v1/fetch/quote` before running it, as a standing team habit, not a one-time check.
- The redesigned system needs roughly 6 to 10 ordinary fields per dam location, all at 1 credit each, and explicitly zero 300-credit parcel fields, since the product never needs ownership, APN, or zoning data.
- Testing against 50 dam locations across 2 to 3 pilot states costs on the order of a few hundred credits for the Mireye layer, against a 25,000-credit monthly budget per person and 75,000 across the team.
- Batch every call through `/v1/fetch/batch` rather than looping per dam, and stay under the 60 requests per minute limit by design, not by accident.
- Divide labor by data layer across the three accounts (one person owns Mireye retrieval, one owns NID plus USGS plus RMA joins, one owns the agent and UI) so the same coordinates are not fetched twice from separate accounts.
- Honest note: credits will very likely not be your constraint on this redesign. Time and the Part 7 agent-design work are the real constraints, which changes where the team should spend its remaining days.

---

## Part 16: Recommended architecture

```text
User (dam-safety analyst) asks a region-scoped question
        |
Agent Orchestrator (LLM-driven intent parsing, genuinely dynamic)
        |
Case Universe Selector (deterministic NID query for the named region)
        |
Tool Planner (decides field set and radius per dam, can widen on thin evidence)
    |-- Mireye (structures, elevation, slope, land cover)
    |-- USACE NID (hazard class, condition, EAP status)
    |-- USGS (nearest-station trend)
    |-- RMA (county drought indemnity)
        |
Enrichment Engine (fixed formula, Part 9)
        |
Sufficiency Check (rule-based first pass, agentic override on borderline evidence)
        |
Escalate / Abstain (bounded retry, capped at 1-2)
        |
Ranking and Decision (agentic: comparability judgment, not just a sort)
        |
Evidence Ledger (field, source ID, timestamp per claim)
        |
Ranked, action-oriented output
```

| Component | Deterministic or agentic | Failure mode to design for |
|---|---|---|
| Intent parser | Agentic | Misreads a region name or an ambiguous ask, needs a confirm-back step |
| Case universe selector | Deterministic | Region name does not map cleanly to a state or county boundary |
| Tool planner | Hybrid | Over-widens radius and burns credits without diminishing returns check |
| Enrichment engine | Deterministic | Silently produces a number from partial data without a confidence tag |
| Sufficiency check | Hybrid, this is the part to build carefully | Rule-based check excludes a strong case for a marginal, technically-correct reason |
| Ranking and decision | Agentic | Ranks two low-confidence dams as if they were equally trustworthy |
| Evidence ledger | Deterministic | A claim in the output with no traceable source, the one failure this design cannot tolerate at all |

---

## Part 17: What to delete

- **The "Risk Disclosure Memo" framing entirely.** It is the single biggest textual signal that pulls this toward Property Diligence.
- **The dual-branch structure as two named products.** Fold into one compound signal per Part 5.
- **The extensive Phase I ESA and CERCLA discussion**, useful for a real business but not for a technical write-up whose brief explicitly asks for question, datasets, enrichment, evaluation, and limits, nothing else.
- **The finance, partnership, and future-expansion sections**, for the same reason. Keep them in an internal document for your own planning. They should not occupy space in the few pages the brief asks for.
- **Subscription alerts and the state-by-state legal appendix**, both future-facing business features with no bearing on whether this satisfies the brief.
- **The duplicate 4.1/4.3 section numbering**, a small fix, but fix it.

None of this work is wasted. It is a genuinely solid business case for a real product. It is simply not what this specific judge rubric is scoring.

---

## Part 18: What to add, ranked by impact

1. **A third, unconventional dataset that does real computation** (Part 10). Without this, nothing else matters.
2. **A genuinely non-hardcoded decision point** (Part 7, Part 6.4 steps 7 to 9). Without this, the "agent" label does not survive a second look.
3. **An actual run of the 10 to 20 case evaluation, with honest failure reporting** (Part 14). This is buildable in a day once the case universe is pulled from NID.
4. **A ranked-list UI that shows the reasoning per item** (Part 12), replacing the memo as the primary surface.
5. **One visible abstention moment in the demo** (Part 13). This single element does more to prove agentic behavior than any amount of architecture explanation.

---

## Part 19: The pitch, rewritten

**10-second pitch:** A ranking agent that tells state dam-safety offices which high-hazard dams to prioritize, because it can see something NID and USGS cannot see alone: which of their dams sit above land that is already economically water-stressed.

**30-second pitch:** Every high-hazard dam in America already carries a federal hazard label. What that label cannot tell a budget-constrained state office is which of those dams would be catastrophic twice over, once from the flood, and again because the farmland below it has no water left to recover on. We join Mireye's structure and terrain data with NID's hazard records and USDA's crop-insurance data to compute that compound number, and rank a state's dam list by it.

**1-minute judge explanation:** covers the question (Part 6.1), the three-source join and why none of them state the answer alone (Part 9), the specific non-hardcoded decision the agent makes when evidence is thin (Part 6.4, steps 7 to 9), and the honest evaluation result once it exists (Part 14).

**Problem statement:** stated in Part 6.1.

**Research question:** does joining physical exposure data with realized agricultural economic loss produce a dam-priority ranking that differs meaningfully from ranking by NID hazard label alone, and does that difference track which dams actually received emergency attention historically.

**Novelty statement:** the join between federal dam-safety data and county-level crop-insurance loss data has not been built before, and it is the crop-insurance layer, not the dam or aquifer data alone, that turns two already-published labels into one genuinely new signal.

**Mireye integration statement:** Mireye's structure and terrain data is the only input in the pipeline that is not already a government-issued risk label, which makes it the part of the computation nothing else in the pipeline can replace.

**Agent statement:** the system decides, per dam, whether its evidence is strong enough to rank confidently, escalates or abstains accordingly, and ranks the remainder by a compound signal it derives itself, not one it was told in advance.

**Enrichment statement:** the exact formula in Part 9, in one line: exposure, from Mireye structures weighted by NID's own hazard categories, multiplied by a water-stress signal built from USGS trend and RMA's realized drought losses.

**Evaluation statement:** tested against every high-hazard, poor-condition dam in 2 to 3 pilot states, checked retrospectively against FEMA's own HHPD grant award history, reported honestly including where it missed.

---

## Part 20: Final verdict

### Current Hydro-Sentinel

| Dimension | Score |
|---|---|
| Challenge alignment | 3/10 |
| Novelty | 3/10 |
| Mireye integration | 3/10 |
| Agentic behavior | 2/10 |
| Enrichment quality | 2/10 |
| Product quality | 5/10 |
| Technical credibility | 7/10 |
| Evaluation readiness | 2/10 |
| Demo strength | 3/10 |
| **Shortlisting probability** | **LOW** |

### Redesigned Hydro-Sentinel (Compound Dam-Rehabilitation Priority Agent)

| Dimension | Score |
|---|---|
| Challenge alignment | 8/10 |
| Novelty | 7/10 |
| Mireye integration | 8/10 |
| Agentic behavior | 7/10, contingent on actually building Part 6.4 steps 7 to 9 as genuinely dynamic |
| Enrichment quality | 8/10 |
| Product quality | 7/10 |
| Technical credibility | 7/10, same strong domain research, better targeted |
| Evaluation readiness | 6/10, design is sound, still needs to be executed |
| Demo strength | 8/10 |
| **Shortlisting probability** | **MEDIUM to HIGH**, depending on execution of the sufficiency and ranking logic, not on anything else in this document |

**The single biggest change you need to make: stop scoring individual parcels for a buy or don't-buy decision, and start ranking a known set of dams for a resource-allocation decision, using a third dataset (RMA crop-insurance loss data) that makes Mireye's fields do real arithmetic instead of decoration.**

### If you have 7 days, in this order

1. Pull the complete NID list of high-hazard, poor-condition dams for 2 to 3 pilot states. This is your case universe and your evaluation set, for free.
2. Lock the Mireye field list via `/v1/meta/fields`, quote a test batch, and download the RMA county drought-indemnity data for the same states.
3. Build the retrieval and Part 9 enrichment formula, then run it across the full case universe.
4. Build the sufficiency and ranking logic (Part 6.4, steps 7 to 9) as genuinely dynamic, not a second pass of hardcoded thresholds.
5. Check the ranking against FEMA HHPD award history, write the honest result, build the minimal ranked-list UI, and rehearse the demo script in Part 13.

### If you have 48 hours

1. Get the NID high-hazard dam list for one pilot state plus Mireye structure and terrain data for each, which alone gives you the exposure half of the signal.
2. Add the RMA join, even in a simplified form, so the compound signal genuinely exists and is demoable, not merely described.
3. Build the smallest possible ranked list with one live abstention case, and rehearse showing the agent decide, not just display.
