# nexus-ontology
A dependency-free Python project that models the core loop of a data-integration
platform: **messy sources -> resolved entities -> semantic ontology -> graph analytics -> explainable risk findings.**

Everything (string metrics, clustering, PageRank, Tarjan SCC, Brandes betweenness,
label propagation) is implemented from scratch using only the standard library.

## Run it

```bash
python -m nexus demo --explain --out out/
python -m unittest discover -s tests -v
```

Requires Python 3.10+. No `pip install` needed.

## What it does

| Stage | Module | Technique |
|---|---|---|
| Ontology | `ontology.py` | Typed objects and links; every write is schema-validated |
| Entity resolution | `resolution.py`, `similarity.py` | Multi-key blocking, Jaro-Winkler / Levenshtein / Soundex, nickname canonicalisation, evidence-weighted scoring with missing-field handling and veto rules, union-find clustering |
| Graph analytics | `graph.py` | Weighted PageRank, iterative Tarjan SCC, Brandes betweenness, label-propagation communities |
| Provenance | `lineage.py` | Every derived entity and score traces back to the source records that produced it |
| Pipeline | `pipeline.py` | Lifts resolved data into the ontology, ranks entities by explainable risk |
| Test data | `synth.py` | Seeded generator: typos, nicknames, format drift, missing fields, a planted laundering ring, structuring, and benign decoys |

## Example output (seed 7, 300 people)

```
source records : 612  ->  resolved entities: 304  (true people: 300)
blocking       : 621 comparisons instead of 186,966  (99.7% reduction)
resolution     : precision 1.000  recall 0.988  F1 0.994
ring detection : raw record graph found 0 ring(s); resolved graph found 1  (recall 100%, precision 100%)
```

### Why entity resolution matters here
The planted money-laundering ring is **invisible in the raw data**: each person
holds accounts under 2-3 slightly different records, so the cycle of payments
is fragmented across nodes. After resolution the cycle closes and is detected.

## Design notes and honest limitations

- **Naive cycle detection over-merges.** Running SCC on the full transaction graph
  fused ~19 unrelated entities into one component, because benign refunds create
  back-edges. The fix: search for rings only on *material* flows (`min_flow`).
- **Blocking trades recall for speed.** Records that disagree on every blocking key
  are never compared (recall 0.988 here, not 1.0).
- **Scoring weights are hand-set.** A natural next step is learning them
  (e.g. Fellegi-Sunter / logistic regression) from labelled pairs.
- **Data is synthetic.** Results show the system works as designed, not real-world accuracy.

## Ideas for extension
- Learn match weights with EM (Fellegi-Sunter)
- Temporal motifs (flow retention along a cycle, time-ordered hops)
- Incremental resolution as new records stream in
- Persist the object store to SQLite and add a query language over the ontology

## Layout

```
nexus/        library code
tests/        21 unit and end-to-end tests
.github/      CI (runs tests on every push)
```

MIT licensed.
