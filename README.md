README — Task 7: Activation & Onboarding Funnel Optimization

Sprint: B - Growth & Experimentation · Phase 3 · PlaceMux AI/ML Engineer track

What this task builds
A cold-start recommendation strategy for new users with no history
Measured lift in first-session relevant actions
A fallback that is never empty
Approach chosen — and what I rejected
Content-based (skill overlap) vs. popularity-based cold start → content-based, primary. Pure popularity shows the same screen to every new candidate regardless of who they are — a candidate with a narrow, specific skill set (deep learning + statistics) would see the same generic top-10 as someone with no listed skills at all. That doesn't just under-personalize, it doesn't try to. Content-based uses what the candidate already provided at signup, with popularity blended in as a supporting signal and safety net, not the primary driver.
Explicit onboarding questions vs. inferring from profile → infer from the profile. Candidates already provide skills at signup (Phase 2's 8-skill ontology). A separate onboarding quiz adds friction between signup and first value shown — directly working against the thing this task is trying to improve.
Strategy design

Blended score = 70% content (skill-overlap) + 30% popularity, ranked descending, with 2 of 10 list positions reserved for weighted-random exploration outside the top-ranked set — per the core concept, this is what lets the system learn a new candidate's taste beyond their self-reported skills, so the second session can already start personalizing.

Measuring lift — honestly, including where it doesn't help

No real online experiment was available, so lift is measured the same way Task 1 handled the offline/online gap: a modeled ground-truth relevance function based on skill overlap, with candidates' self-reported skills treated as an incomplete (not wrong) proxy for their true skill set — realistic noise, not a fantasy of perfect self-report.

Segment	Baseline (popularity-only)	Cold-start strategy	Lift
Overall	69.6%	90.9%	+30.6%
Candidates with skills on file	72.2%	95.1%	+31.8%
Candidates with no skills on file (7.8%)	39.3%	41.9%	+6.5%

The near-parity result for the no-skills segment isn't a shortfall to hide — it's expected and correctly reported: when there's no skill signal, the cold-start strategy becomes popularity-based itself (see fallback logic below), so the two approaches largely converge. The small residual lift comes from the exploration slots still operating even in that fallback mode.

The never-empty fallback — verified across every realistic layer, not just the happy path
Scenario	Recs returned	Strategy used
Normal candidate, full job data	10	content_popularity_blend
No skills on file, full job data	10	popularity_fallback_no_skills
Candidate's skill matches zero jobs	10	content_popularity_blend (popularity component keeps it non-empty)
Forced failure: job catalog completely empty	0	static_fallback

The last row is deliberately shown as a real limit, not smoothed over: when there are genuinely zero jobs to recommend, no ranking strategy can invent postings that don't exist. That's not a bug in the recommender — it's the actual floor of what "never empty" can mean. The correct fix lives one layer below this task's scope: the onboarding UX should detect zero available postings and show a distinct "we're launching in your area soon" state, rather than the recommender silently returning nothing. Flagged as a hand-off item to Frontend, not solved here, since it's a product/UX decision, not a ranking one.

Worked example (fresh account, live)

New candidate enters [python, machine_learning, statistics] at signup. Top recommendation: job_599 at a blended score of 0.776, with the explanation shown to the user: "Ranked primarily by overlap with your listed skills (python, machine_learning, statistics), blended with a popularity signal."

Known limitations (disclosed)
Lift is simulated, not from a real A/B test — the direction (content-based beats popularity-only when skills exist) should transfer, but the exact 30.6% figure needs re-validation against a real online experiment before being used as a launch metric
The 8% no-skills-on-file rate is an assumed onboarding drop-off, not measured from real signup funnel data in this environment
Exploration slot selection is a simple weighted-random sample, not a formal multi-armed-bandit policy (e.g. Thompson sampling) — sufficient to demonstrate the core concept, but a production implementation should use a policy that properly balances explore/exploit
Files
File	Contents
p3_task07_cold_start.py (as .txt)	full pipeline, self-contained
p3_task07_cold_start_results.csv / _baseline_results.csv	per-candidate first-session simulation results
p3_task07_worked_example.json	fresh-account demo with explanation
p3_task07_never_empty_verification.json	all 4 fallback layers, including forced failure
p3_task07_summary.json	consolidated results + limitations
p3_task07_lift_and_fallback.png	lift-by-segment chart + never-empty verification chart
