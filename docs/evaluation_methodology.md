# **Evaluation Methodology v2 — Trajectory & Recipe Graphs**

QCRI · StoPlan / RoBee · single extractor (GPT-5.1) · reference-free · judge \= DeepSeek

This version supersedes the v1 deck methodology on four points, all driven by what the extracted corpus actually looks like: (1) full per-criterion label storage, (2) the recipe `primitive` key, (3) legal Repeat cycles, and (4) a descriptive layer that runs before anything else and lives in the same spreadsheet as the data.

## **Pipeline (per link)**

Descriptives → Mechanical → Judge → Compose → Store. Steps 1, 2, 4, 5 are deterministic code; step 3 is the only model cost, and the judge (DeepSeek) is never the extractor (GPT-5.x).

### **Layer 0 · Descriptive analysis (tab: `descriptives`)**

Runs over every graph in both source tabs, one output row per graph, before any evaluation. Stored per graph: sizes (nodes, edges, density), degrees (max in/out, isolated nodes, dangling edges, duplicate ids), cycle structure (has\_cycle, back-edge count, legal\_repeat\_cycle), depth (longest path on the back-edge-free graph, linearity \= longest path / edges), grounding surface (evidence coverage, average evidence length), calibration surface (certainty stated/implied counts, unknowns count), semantic composition (full node-type counts as JSON; start\_state / outcome / retry counts for trajectories; Op/Branch/AND/OR/NOT/Repeat counts, ungated edges, Repeats missing stop\_condition, Branch arity/fallback issues, personal-log-leak node count for recipes), and the gate flags the graph would trigger. This layer answers "what does the corpus look like" and pre-screens which graphs will hard-fail before spending judge tokens.

### **Layer 1 · Mechanical metrics (computed inline at evaluation time)**

Per graph, against the fetched article text: EGR (verbatim-grounding rate, normalized matching), HNR (1 − EGR), plus the structural checks above. These populate the hard-fail gates and are given to the judge; the judge applies them but never recomputes them.

**Schema alignment (new in v2).** Recipe node types are read from the `primitive` key (with `type` as fallback). Untyped nodes are counted, not crashed on.

**Cycle semantics (new in v2).**

* Trajectory: any cycle is a hard fail (lived time cannot loop).  
* Recipe: a cycle is LEGAL iff every back-edge leaves a Repeat node that has a stop\_condition; the loop-back is exactly the LA-CFG Repeat construct. Only cycles failing this test trigger the `cycle_not_repeat_gated` gate. Legal loops must not depress logical or structural scores.

Hard-fail gates, trajectory: invalid schema · HNR\>0.20 · any cycle · start\_state count ≠ 1 · missing outcome · logic-edge leak. Recipe: invalid schema · ungrounded\>0.20 · non-Repeat-gated cycle · Branch missing fallback or arity\<2 · Repeat missing stop\_condition · personal-log leakage. Judge-added gates: systematic advice-as-step leakage; hindsight-as-fact (trajectory); overlapping or non-covering Branch predicates (recipe).

### **Layer 2 · The judge (DeepSeek, prompt\_judge\_deepseek\_v2.txt)**

Reference-free, evidence-anchored, length-neutral form-filling. For every criterion the judge must return four fields: integer score 0–4, a verbatim article span (≤30 words), the node/edge ids the score bears on, and a ≤20-word note. "No quote, no score." Off-type attempts are judged for restraint (over\_reach true/false with worst offending nodes); on-type attempts get over\_reach=null. Cross-graph coherence (consistency .40, clean\_split .30, alignment .30) is scored only when both graphs have ≥3 grounded nodes.

Rubrics and weights are unchanged from v1:

* Trajectory (TQS): grounding .30 · coverage .20 · temporal .20 · structural .10 · granularity .10 · calibration .10.  
* Recipe (RQS): grounding .25 · coverage of procedure .20 · logical correctness .25 · structural .10 · granularity .10 · calibration .10.  
* TQS / RQS \= 100 × Σ wᵢ·(sᵢ/4), gates applied first and never averaged away.

### **Layer 3 · Composition and verdicts**

A graph is "good" when it passes ALL gates AND composite ≥ 75 AND grounding criterion ≥ 3.6/4 (equivalently EGR ≥ 0.9). A link is well-handled when every on-type graph is good, no off-type attempt over-reaches, and coherence ≥ 75 where applicable.

### **Layer 4 · Storage (tab: `evaluation`) — full labels, new in v2**

One row per link. Nothing is collapsed into a bare composite; every rubric label is a column:

* identity: link, class (trajectory / recipe / mixed by bank membership), article\_chars, status;  
* trajectory axis: t\_grounding\_faithfulness, t\_coverage, t\_temporal\_correctness, t\_structural\_validity, t\_granularity\_normalisation, t\_calibration\_abstention (integer scores), t\_criteria\_notes (JSON of note+span+ids per criterion), TQS, t\_gate\_flags, t\_passed\_gates, t\_over\_reach, traj\_good;  
* recipe axis: the mirrored r\_\* columns with coverage\_of\_procedure and logical\_correctness, RQS, r\_gate\_flags, r\_passed\_gates, r\_over\_reach, recipe\_good;  
* coherence axis: c\_consistency, c\_clean\_split, c\_alignment, coherence, coherence\_reason;  
* link verdict: link\_well\_handled, worst\_offenders;  
* provenance: mech\_json (the exact mechanical input the judge saw) and judgement\_json (the full judge output, spans included), both capped to the sheet cell limit.

This makes every composite auditable: for any surprising TQS/RQS you can read which criterion moved, on what evidence span, over which nodes — without rerunning anything.

### **Reliability guardrails (unchanged)**

Judge ≠ extractor · evidence-anchored scoring · length-neutrality · double-run option (rerun the judge and flag any criterion differing by \>1 point; per-criterion storage makes this comparison a column diff) · human spot-check of 5–10 graphs.

### **Honest limits (unchanged)**

No gold graphs, so no node/edge-F1, no Structural Hamming Distance, no measured recall of true salient events; judged coverage/correctness are estimates against the text. A small gold set can be added later to unlock F1/SHD.

### **Reporting**

From the two tabs, the dataset scorecard aggregates: per-class means and medians of TQS/RQS and every criterion column, gate-failure rates by gate, over-reach rate per extractor direction, coherence distribution on mixed links, and the worst-offender list. Criterion-level columns also enable the diagnostic cut v1 could not produce: e.g. "recipes lose most of their points on logical\_correctness (fallback edges), not on grounding," stated directly from column medians.

