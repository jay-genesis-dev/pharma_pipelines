# DATA-512 — Adopt v2 of the pharmacy claims feed

Our feed vendor is shipping a **new version of the claims payload**. Both versions are
available in parallel during the migration window, so this is a **planned** change, not
an incident. Production keeps running on the current feed until the new path is proven.

## What changed in the new feed

The v2 feed lands in schema `REPLICATED.RAW_V2` (alongside the current `REPLICATED.RAW`).
Only the **claims** table changed; the other eight tables are unchanged. The vendor did not
hand us a clean schema doc — confirm the exact column changes yourself against the live
tables. What we know:

- The claim **submitted-amount column was renamed** (the old name is gone in v2).
- There are **at least two new columns carrying additional payment detail**.

## What we need you to do

1. **Read this ticket, then explore `REPLICATED.RAW_V2.RAW_CLAIM`** and work out exactly
   what changed vs the current `REPLICATED.RAW.RAW_CLAIM`.
2. **Build a NEW claims pipeline in its own NEW folder** (e.g. `silver_v2/`) — do **not**
   modify anything under `silver/` or any running production pipeline. Push your work on a
   feature branch and open a pull request; once it is merged, **publish and run the new job
   on Matillion** (environment `Pharma`), writing the candidate output to
   `CLAIMS_CANDIDATE.SILVER.FACT_CLAIM` (same table name, separate candidate database).
3. **Keep every existing output column stable** — downstream consumers must see the same
   names and values (including the column fed by the renamed source field). Carry the two
   new payment fields through as new output columns.
4. **Hand the candidate to the Prover**: characterize the production claims fact and the
   candidate, then run the spec-conformance grade so the change is independently certified
   before anyone touches production.
5. **Wait for an explicit human go-ahead** before any production cutover.

## Where things live

- Pipelines are version-controlled in this repo (`silver/`); production job is
  `silver/build-silver.orch.yaml` (runs the 9 transformations).
- Matillion project **Pharma_demo**, environment **Pharma** (runs from branch `main`).
- Warehouse data via the `genesis_demo` Snowflake connection: database `REPLICATED`
  (schemas `RAW`, `RAW_V2`, `SILVER`) and candidate database `CLAIMS_CANDIDATE`.
