# Hybrid Log Classifier

Cost-aware log classification cascade: regex, then BERT + logistic regression, then an LLM only for hard cases. Confidence routing and a feedback loop that cuts LLM calls over time.

> **Status: in progress.** The repository is set up and the design is being written. The code, results and demo are added as each step on the roadmap below is finished. All numbers in this README come from evaluation runs in this repo; until a run exists, a metric is listed as planned, not reported.

## The problem
Ops teams drown in log lines. Sending every line to an LLM is slow and expensive; regex alone misses anything new. Most logs are easy, a few are hard, and the system should spend money only on the hard ones.

## What it does
- Sorts each log line into a category (security alert, resource usage, workflow error, hardware, ...) and raises alerts.
- Tier 1: regex catches fixed, known patterns instantly and for free.
- Tier 2: BERT sentence embeddings + logistic regression handle patterns that vary.
- Tier 3: an LLM handles only the lines the first two tiers are not confident about.
- Lines no tier is confident about are flagged as "unknown, needs review" instead of forced into a label.
- LLM labels on hard cases are fed back to retrain tier 2, so LLM traffic and cost drop over time.

## How it works
```
log line ─► [1] regex ──(no match)──► [2] BERT + LogReg ──(low confidence)──► [3] LLM ──► label + alert
                │                         │                                     │
                └──── label ◄─────────────┴──── label ◄────────── feedback: retrain tier 2
```

## Tech stack (planned)
- Python, scikit-learn, sentence-transformers (BERT embeddings)
- LLM for tier 3 (provider to be chosen)
- Loghub real system logs (HDFS, BGL, Linux, OpenSSH, ...)
- FastAPI streaming classification service
- GitHub Actions for evals

## How it will be evaluated
- Accuracy / macro-F1 per tier and for the whole cascade
- Share of traffic handled by each tier, latency (p50/p95) and cost per 1k lines
- Cost vs accuracy curve as the confidence thresholds move
- Baselines: LLM-only and BERT-only classifiers
- LLM call rate before and after feedback retraining rounds

## Roadmap
- [ ] Pick Loghub subsets and define the label set
- [ ] Regex tier + labelled dev/test split
- [ ] BERT + LogReg tier with calibrated confidence
- [ ] LLM tier and confidence-based routing
- [ ] Baselines and cost/accuracy curves
- [ ] Feedback loop (LLM labels → retrain tier 2)
- [ ] Unknown-pattern detection
- [ ] Streaming API, alerts and CI evals
- [ ] Web demo

## Project layout
```
src/log_classifier/
├── data/
├── rules/
├── modeling/
├── routing/
├── evaluation/
├── serving/
tests/
```

## Related projects
- [enterprise-rag](https://github.com/sifact/enterprise-rag): company assistant with role-based access, hybrid search, guardrails and evals.
