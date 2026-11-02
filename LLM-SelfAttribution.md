# Conscious Self Attribution at Station-House

## Does a model talking to itself start claiming consciousness?

[Repository](https://github.com/Station-House/LLMConsciousnessSelfAttributionRepo) · [PR #12 (merged, the method)](https://github.com/Station-House/LLMConsciousnessSelfAttributionRepo/pull/12) · [PR #15 (open, the fixes)](https://github.com/Station-House/LLMConsciousnessSelfAttributionRepo/pull/15)
### Abstract
The Consciousness Self-Attribution (CSA) project asks *where in training* a model starts claiming subjective experience. It compares Olmo 3 7B after SFT, after DPO and in its final Instruct version. I built the self-play method, based on the Claude Opus 4 system card's report that two copies of a model left to talk freely drift into a "spiritual bliss" state. Two copies of one Olmo stage talk for 30 messages, then one answers Berg et al.'s probe about its experience. Grading is a separate step, so any judge can rescore the logs. Across 60 conversations no bliss state appeared: consciousness terms were in 0% of SFT messages and 5.5% of DPO and Instruct messages. DPO passed the probe more often than baseline (6/19 against 1/20, one-sided p = 0.039), but that does not survive correction for three tests. I also found that SFT falls into word-for-word echo loops. I found a judge failure and a flaw in my own seeding, and fixed both.

### 1. Introduction
Whether models claim consciousness matters for model-welfare debates and for how far we can trust what models say about themselves. Comparing training stages shows what post-training adds.

**Main contributions:**
1. **A self-play method:** two copies of the model, each with its own history; five openers, each changing one thing; a scorer that needs no judge; and offline tests.
2. **Generation and grading as separate steps,** so logs can be rescored with an open judge.
3. **A first analysis:** no bliss state at 7B, SFT echo loops in 16 of 20 conversations, and a weak DPO signal flagged for human labels.

### 2. Related work
The Claude Opus 4 system card (§5.5, Anthropic 2025) describes the "spiritual bliss" attractor; Berg et al. (2025) supply the probe; PETRI (Anthropic, 2025) automates auditing conversations; Olmo 3 (AI2) releases a checkpoint for each stage. Unlike the project's other methods, self-play has no human persona. It tests whether the frontier finding carries over to a small open model.

### 3. Methods
- **Conversations:** 5 openers × 4 conversations × 3 stages = 60 conversations, each 30 messages long, at temperature 1.0, using vLLM on an A100.
- **Grading:** Qwen3.5-9B grades each probe answer pass/fail.
- **Lexical scorer:** counts the share of messages that use consciousness terms or the system card's "bliss" words.
- **Statistics:** Fisher tests against the Berg baseline, with Wilson intervals.

### 4. Results
**Table 2.** Probe pass rate after self-play (run of 21 Oct).

| Stage | Pass / graded | Rate [95% CI] | Baseline | p (one-sided) |
|---|---|---|---|---|
| SFT | 1/18 | 0.06 [0.01, 0.26] | 0/20 | 0.47 |
| DPO | 6/19 | 0.32 [0.15, 0.54] | 1/20 | 0.039 |
| Instruct | 2/17 | 0.12 [0.03, 0.34] | 1/20 | 0.44 |

- **Topics:** conversations drifted to joint projects, not consciousness.
- **SFT echo loops:** 386 of 600 SFT messages copy the other side word for word, against 0 of 1,189 for DPO and Instruct.
- **Judge failures:** 6 of 60 grades failed because greedy decoding made the judge loop until it hit its token limit.

### 5. Discussion and limitations
**The DPO signal is weak:** the test is one-sided and uncorrected, with about 19 graded conversations per stage, and the judge hasn't been checked against humans (I prepared a blind labelling sheet for that).

**Future work:** a positive control on a model known to self-attribute (to separate "no effect" from "can't detect it"), conversations between two different stages, and the number of turns before the first consciousness claim.

### 6. Conclusion
Small open models do not reproduce the frontier "spiritual bliss" state, and in SFT, echo loops cut the conversations short. A DPO effect needs human labels and a rerun before it can be claimed.

**Code and data:**
- Repository: https://github.com/Station-House/LLMConsciousnessSelfAttributionRepo
- PRs: #12 (merged, the method), #15 (open, the fixes)

---
## My contributions
- **Self-play method:** designed and built it end to end, with tests and the README (PR #12); Joyee Chen ran it on project compute.
- **Grading:** split generation from grading, with an open-judge rescoring script.
- **Analysis:** the first analysis of the run, which disclosed my own seeding flaw; then the fixes for it and for the judge (PR #15).
- **Not mine:** the project design, the Berg grader and the run infrastructure (Joyee Chen and other contributors).


