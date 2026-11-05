# Submission checklist

The official problem statement lists no fixed deliverables. This maps its rubric to evidence.

| Rubric criterion (points) | Where the evidence is | Status |
|---|---|---|
| Research novelty (15) | Report §3, §10; coupling-aware fusion; agent evaluated as an alert policy | Done |
| Technical integration: agentic AI (13) | `forestguard/agent.py`, report §5.5, audit log, 22 tests | Done |
| Technical integration: signal processing (12) | `audio_dsp.py`, `vibration_dsp.py`, report §5.1–5.3, Figure 9 | Done |
| Experiments and evaluation (20) | Report §9: baselines, ablations, sweeps, latency; `results/` | Done |
| Problem significance (10) | Report §2 | Done |
| Research potential (15) | Report §12–13, next research question | Done |
| Prototype evidence (10) | Dashboard, screenshots, `example_audit_log.jsonl`, CSV exports | Done |
| Reproducibility (5) | README, Colab notebook, `run_all.py`, fixed seeds, requirements | Done |

## Files
- [ ] Source code zip (`forestguard_source.zip`)
- [ ] Word report (`FORESTGUARD_Technical_Report.docx`) and PDF
- [ ] Slides (Claude Slides; export to PPTX or PDF)
- [ ] Pitch, demo script, Q&A (`PITCH_AND_QA.md`)
- [ ] Dashboard screenshots (`screenshots/`)
- [ ] Results and figures (`results/`)

## Before presenting
- [ ] Add team members' names to the report title page and slides
- [ ] Run `python -m pytest -q tests` live if asked (expect 22 passed)
- [ ] Optional in Colab: YAMNet run (`--backend yamnet`) and FSC22 cross-dataset test; add the numbers only if run
- [ ] Every vibration number is said aloud as "simulated"
