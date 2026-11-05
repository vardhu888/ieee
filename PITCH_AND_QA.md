# FORESTGUARD AI — pitch, technical explanation, demo script and judges' Q&A

All numbers come from `results/results_summary.json`. Say "simulated" whenever a number involves vibration.

## 60-second pitch
Forest acoustic monitors don't fail because they miss chainsaws. They fail because thunder, wind and grazing
cattle sound like threats, and rangers stop trusting them. In our simulation, a stateless detector alerting on
every suspicious window raises **79.0 false alerts per hour** of quiet forest.

FORESTGUARD treats a loud sound as a claim, not a verdict. A geophone asks whether the ground agrees. A stateful
agent asks whether the evidence persists, wakes the vibration sensor only when audio is suspicious, and explains
every decision in a tamper-evident log.

On 800 simulated monitoring episodes built from held-out real recordings, the agent cuts false alerts to
**6.2 per hour** while still catching **79%** of intrusions, and
98% of logging events. Vibration evidence alone accounts for a **1.9×** reduction. The audio
model is validated on real data; the vibration channel is simulated, and we say so. The next step is a real
geophone pilot. Everything runs in about 38 ms per 5-second window, so it fits on a solar-powered node.

## 3-minute technical explanation
1. **Problem (20 s).** Decide every 5 seconds whether to keep monitoring, gather more evidence or call a ranger,
   using a microphone and a ground-vibration sensor, under noise, missing data and natural look-alikes.
2. **Signal processing (40 s).** Audio: 80 Hz high-pass, energy segmentation against the clip's own noise floor,
   log-mel/MFCC features, onset cadence and a harmonic-product fundamental, feeding a 22-class model with
   68.8% accuracy (real ESC-50, official folds). Vibration: RMS and peak SNR, Welch FFT, dominant frequency,
   Hilbert envelope, peak detection and gait-cadence autocorrelation, feeding a source-family classifier.
3. **Simulator honesty (30 s).** No paired forest dataset exists, so we simulate the geophone with physics:
   distance attenuation that kills high frequencies, natural sources that also shake the ground, unrelated animals,
   hum and dropout. Footstep timing and engine frequencies come from the real recording. Every file says SIMULATED.
4. **Fusion (30 s).** Bayesian log-odds. The likelihood ratio for each threat type is learned on training folds,
   so "no vibration" counts heavily against a vehicle but barely against a voice. A frequency-match check
   confirms that sound and vibration share one source.
5. **Agent (40 s).** Context, active sensing, sensor health, decaying belief and 30 s memory. It escalates on
   corroboration or persistence and raises the bar when expected corroboration is missing. One alert per incident,
   cooldown, and a SHA-256 chained log with reasons.
6. **Evidence (20 s).** Stateless audio: 79.0/h. Agent: 6.2/h at 79% recall.
   Same agent without vibration: 12.0/h. Development and test draws are separate, ablations are fair,
   and sweeps show the false-alarm benefit survives weak coupling. Heavy dropout costs recall, and we show that too.

## 5-minute demonstration script
| Time | Do | Say |
|---|---|---|
| 0:00 | Dashboard header, KPI tiles | "One node, 5-second windows. Real audio badge, simulated vibration badge, always visible." |
| 0:30 | Sidebar → scenario **logging**, fold 1 | "A 60-second patrol from held-out clips: chainsaw windows inside background sound." |
| 0:50 | Live monitor timeline | "Blue is audio-only threat probability, orange is fused. Red bands are the true intrusion. The dashed line is the alert." |
| 1:30 | Alert card | "Every alert says why: type, confidence, corroborated or persistence-only." |
| 1:50 | Tab **Signals**, choose a chainsaw window | "Real spectrogram with detected segments, the classifier's top five, the simulated geophone and its FFT. The dominant frequency is compared with the audio fundamental." |
| 2:40 | Cross-correlation plot | "Sound and ground envelopes line up in time: independent evidence of one source." |
| 3:00 | Scenario **storm** | "Thunder is loud and shakes the ground. The agent recognises a weather pattern and does not escalate." |
| 3:30 | Dropout slider to 0.6, re-run **foot_patrol** | "Vibration keeps dropping out. The agent records each gap and flags the sensor for maintenance. Our sweeps show this is our weakest case: at heavy dropout it misses more than a no-vibration agent, and switching policy on that flag is the next fix." |
| 4:00 | Tab **Agent decisions** | "The hash-chained log: reasons, evidence terms and hashes. Edit one record and verification fails; that's a unit test." |
| 4:30 | Tab **Experiments** | "Real audio metrics, simulated fusion and episode results, and the trade-off plot. Then limits: the vibration is simulated, and brief single-window presence is often not escalated." |

## Likely judges' questions
**Q: Isn't fusion trivially good because you simulated the vibration from the labels?**
A: We designed against that. Natural sources also produce vibration (thunder, wind, rain, livestock with
human-like cadence). Distance is random, so many threats are invisible to the geophone. Unrelated animals, hum and
dropout are added independently of the label. Vibration alone reaches only F1 0.713 and
recall 0.616. We also report sweeps over sensor strength and dropout rather than one number.
The remaining caveat is that footstep timing and engine frequencies come from the audio, so the frequency-match benefit is
partly built in. We state that in the limitations.

**Q: What is actually real?**
A: Every audio clip, every audio probability (out-of-fold on official folds), all acoustic DSP, and the
68.8% accuracy and 0.784 threat F1. The vibration channel and episodes are simulated.

**Q: Why not a neural fusion model?**
A: A learned late-fusion baseline is in the table (window F1 0.868 against our 0.859).
Our Bayesian form gives per-type likelihood ratios the agent can reason with and explain, and it needs very little data.

**Q: How is this agentic, not just a classifier with a threshold?**
A: It keeps state (belief and history), acts on its own sensing (wakes the geophone, costing energy only when
needed: 55% duty cycle with no loss against always-on), adapts to context and sensor health, changes its evidence
bar depending on what corroboration should exist, and chooses among actions with an explanation for each.

**Q: Did you tune on the test set?**
A: No. v1 was developed on a seed-42 episode draw. Fixes were selected there with a rule fixed beforehand, and every
reported episode result is on a separate seed-2026 draw. That's also why we report the gap: the selected mode gave
4.4/h on development but 6.2/h on test.

**Q: What happens when the geophone fails?**
A: The agent flags maintenance after three missing windows in five, and keeps alerting from audio with a higher evidence bar. Honestly, that bar is too high when the sensor is mostly gone: at 80% dropout recall is 0.57 against 0.71 for a no-vibration agent. Switching to the no-vibration policy when the sensor is flagged is the first fix on our list.

**Q: Why is brief presence detection low?**
A: A single 5-second voice or step is weak evidence. We deliberately require persistence or corroboration, which is what
cuts false alerts. A department that wants that recall can lower k, at a measurable false-alarm cost shown by the ablations.

**Q: Gunshots?**
A: ESC-50 has no gunshot class, so fireworks is a labelled proxy with a single-event escalation rule. Validating it
on real gunshot recordings is future work.

**Q: Real-world cost and power?**
A: Roughly ₹10,000–19,000 per node in indicative parts (MEMS microphone, geophone and ADC, ESP32/RPi-class compute,
LoRa, 10 W solar). Compute is 38 ms per 5 s window on a desktop core, and only events are transmitted.

**Q: Next research question?**
A: Does vibration corroboration still cut false alerts several-fold with real geophones in real soil? A paired pilot
with patrol-log ground truth answers it. After that: multi-node corroboration, localisation and learning
thresholds from ranger feedback.

**Q: Privacy?**
A: Audio stays on the node. Voices are classified only as presence, never recorded or transcribed. Only compact event
messages are sent.
