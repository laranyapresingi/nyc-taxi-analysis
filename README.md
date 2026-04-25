# NYC Yellow Taxi — Data Science Portfolio Project
### Dataset: NYC TLC Yellow Taxi · November 2025 · 4.18M trips
### Stack: Python (pandas, matplotlib, seaborn) · SQL · Tableau
### Scope: Demand analysis · Revenue analysis · Passenger behaviour

## Project Overview
This project analyses one month of NYC Yellow Taxi trip data to answer real operational questions a data scientist at a ride-hail platform (Uber, Lyft) would be expected to answer. The analysis is split across three analytical notebooks, each with a clearly scoped business question and reproducible methodology.
The dataset is publicly available from the NYC Taxi & Limousine Commission (TLC).

## Data Architecture
Three analytical layers are built from the cleaned base dataset and used consistently across all notebooks:
LayerScopeUsed Forlayer1_dfAll vendors, no filterDemand counting, temporal patterns, geographic volumelayer2_dfVendors 1 & 2 only, extended schema completePassenger counts, rate codes, airport/congestion feesrevenue_dfAll vendors, refunds/adjustments excludedAny monetary analysis (fare, total, tip)
The rationale for each layer is documented in-notebook. Vendor 6 (Myle) reports 100% missing values on extended schema fields and is excluded from layer2_df. Negative total_amount values represent financial reversals, not real trips, and are excluded from revenue_df.

## Notebooks
### 03 — Demand Analysis
Central question: Where and when should a ride-hail platform concentrate driver supply to best match passenger demand?
Mismatched supply and demand causes two failure modes: passengers wait too long (demand > supply), or drivers idle without rides (supply > demand). This notebook finds the answer across four dimensions.

### Section A — Temporal Demand Patterns

When is demand highest, and how does it shift across the month?

Finding: Two fundamentally different demand regimes exist in the data. Weekday demand is commuter-driven — a sharp morning peak at 8–9am followed by an evening rise peaking around 6pm. Weekend demand follows a single broad afternoon-to-evening climb that stays elevated well into the night. Weekend demand is ~3x higher than weekday at midnight; weekday demand is ~2x higher at 8am. A platform treating both regimes identically would misallocate drivers in both directions.
A 7-day rolling average over the month reveals a demand decline beginning around Nov 17 — more than a week before Thanksgiving — reflecting reduced business travel and commuter activity as the holiday approaches. By month-end, baseline demand had dropped ~18% from early-November levels.

### Section B — The Thanksgiving Effect

Does a major holiday meaningfully disrupt demand, and in what direction?

Finding: The Thanksgiving demand shock is not symmetric. Thanksgiving Day (Nov 27) records the lowest single-day volume of the month (~32% below a typical weekday). But the surrounding days show demand deviations in the opposite direction — Thanksgiving Eve and the post-holiday weekend show elevated activity. This asymmetric pattern — demand shifting to adjacent days rather than simply disappearing — is the signal a demand forecasting model needs to learn.

### Section C — Geographic Demand

Which zones generate the most trips?

Finding: Manhattan dominates both pickup and dropoff volume, but the composition differs between the two. Zones that appear in the top-20 pickup list but not the top-20 dropoff list (and vice versa) are the zones with structural supply-demand imbalance. Airport zones in Queens generate substantial pickups, but drivers arriving there via a dropoff face queuing time before their next ride — a distinct operational challenge.

### Section D — Supply-Demand Zone Imbalance

Which zones are chronically under- or over-supplied with drivers?

Finding: Imbalance is measured as the difference between pickups (demand signal) and dropoffs (supply arrival signal) at each zone. The output identifies two actionable zone types: deficit zones where passengers are waiting longer than necessary, and surplus zones where drivers are idle. A platform would use this output to design surge pricing triggers in deficit zones and incentive re-positioning bonuses from surplus zones.

### 04 — Revenue Analysis
Central question: What trip characteristics drive revenue per hour for a driver — and what does an optimal shift look like?
Total revenue is a misleading metric. A driver cares about earnings per hour of time invested, not earnings per trip. A long trip might pay more in absolute terms but be worse per hour than two shorter trips in the same window.

### Section A — Effective Hourly Rate by Time Slot

When does driving earn the most per hour?

Finding: Effective hourly rate and trip volume are not the same signal — and that gap is the key insight. The hours with the highest trip volume (evening rush) are not the hours with the highest earnings per hour. Late night and early morning trips are faster (less traffic), so fare-per-minute is higher even though absolute fares are similar. A driver optimising for earnings per hour should prioritise these windows, not simply the peak-demand windows. The genuinely optimal shift window by effective hourly rate may not be the window with the highest demand or the highest per-trip fare.

### Section B — The Airport Premium

How much more do airport trips actually pay, and is the premium real after accounting for trip duration?

Finding: The airport premium is real in absolute fare terms, but the effective hourly rate is the honest test. JFK flat-rate trips have a known fixed fare regardless of traffic, which makes their effective hourly rate sensitive to how long the trip actually takes. If airport trips are long and slow, the $/hr advantage shrinks or disappears vs. multiple shorter city trips in the same window. The tip analysis reveals whether airport passengers tip proportionally better — an additional factor for drivers deciding whether to queue at an airport rank.
Airport trips represent a small share of total volume but a disproportionate share of total fare revenue — the headline finding quantified in the notebook with real numbers from the data.

### Section C — Tip Determinants

What trip characteristics predict higher tips?

Note: This analysis covers credit card trips only, as cash tip amounts are not recorded in the TLC dataset. Cash-trip tip rates are structurally unobservable and are not imputed.
Finding: Three tip determinants are visible across the analysis: trip distance, trip duration, and time of day. The group size panel (using passenger_count from layer2_df) tests whether groups tip differently than solo riders — relevant for any ride-pooling product decision. If fare-per-passenger falls for group trips, the platform is effectively subsidising group travel; if it rises, groups represent a higher-yield segment worth targeting.

### 05 — Passenger Behaviour
Central question: Are there distinct passenger segments with different behaviours and economics — and what do those differences mean for platform design?

### Section A — Payment Method Segmentation

How does payment method vary by time of day and geography?

Note: Cash tip amounts are not captured in the dataset. Payment-type fare comparisons use revenue_df; payment-type share counts use layer1_df to include all vendors.
Finding: Cash share as a percentage of hourly trips is the honest view — raw trip counts are dominated by the overall demand curve and mask the signal. A late-night cash spike would suggest a distinct passenger population or card terminal connectivity issues during those hours. Geographically, cash payment rates are higher in outer boroughs than Manhattan, consistent with a card infrastructure access hypothesis. This is directly actionable: a platform expanding card coverage should prioritise the highest-cash boroughs first.

### Section B — Group vs Solo Trip Economics

Do group trips behave differently than solo trips, and what does that mean for pooling product design?

Finding: The unit economics panel — fare per passenger — is the most important output for product decisions. If fare-per-passenger falls for group trips, the platform is subsidising groups relative to solo riders. Group trip share is not constant across the day; it peaks at specific hours, and if the peak aligns with late evening or weekend nights, it confirms group travel as a leisure-driven behaviour. This identifies the optimal window for a ride-pooling trial or incentive campaign.

### Section C — Data Quality Audit

What did the cleaning process remove, and why?

Finding: The cleaning funnel makes every removal decision transparent and quantified — no rows are silently dropped. The largest single reduction in layer2_df is the vendor scope restriction, which removes Vendor 6 (100% missing extended schema) and null rows from Vendors 1 & 2. This is structural data absence, not corruption, and is documented accordingly.
The vendor-level anomaly breakdown is the most actionable output for a data engineering team:

Vendor 7 (Helix): drives all zero-duration trips
Vendor 6 (Myle): drives all structural missingness on extended schema fields
Vendor 2 (Curb Mobility): concentrates all negative fare values

These are vendor-specific reporting behaviours, not random data corruption. A production pipeline would handle them with vendor-specific logic rather than a single generic cleaning step.
