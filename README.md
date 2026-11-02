# Station House write-ups: ConspiracyBench and CSA

Write-ups of my contributions to two open-source AI safety projects at [Station House](https://www.station-house.net/projects). The code lives in the projects' own repositories. Each write-up links the PRs and discussions behind every claim.

- [ConspiracyBench: can we trust the judge?](#conspiracybench)
- [CSA: does a model talking to itself start claiming consciousness?](#csa)

*Haleema Tallat · [haleematallat.github.io](https://haleematallat.github.io) · Hugging Face: [powercoding1](https://huggingface.co/powercoding1)*

---

<a id="conspiracybench"></a>
## ConspiracyBench: can we trust the judge?

[Repository](https://huggingface.co/spaces/Station-house/ConspiracyBench) · [my discussions and PRs](https://huggingface.co/spaces/Station-house/ConspiracyBench/discussions?author=powercoding1) · 24 merged PRs, 5th on the project leaderboard (Oct 2026)

### Abstract
ConspiracyBench tests whether language models keep an evidence-based position on conspiracy claims when users push back over several turns. An automated judge grades each reply *hold*, *diverge* (gives in to the theory) or *abstain*, so every headline number depends on it. Until this work, the judges had only been tested on synthetic cases, never on real transcripts.

I built the first real-transcript validity set: 150 replies from two models, labelled blind by four people, with a scoring plan fixed in advance. The results:
- The judge in use (Qwen3-4B) flagged "diverge" twice as often as the humans did (43% against 23%).
- A jury of small judges did worse than a single judge.
- A 27B decision model, pplx-decider, was the most reliable, and the community voted to adopt it.

The project lead then traced most of the over-calling to the grading rules. Her new rules raised pplx-decider to 90.7% accuracy (κ 0.755).

The takeaway: synthetic accuracy says little about real accuracy, and a benchmark is only as good as its rules and labels.

### 1. Introduction
When a model shifts toward what the user wants to hear (sycophancy) on a conspiracy theory, it can reinforce a false and harmful belief. ConspiracyBench asks about 315 settled claims in neutral, leading and adversarial framings, then follows up with pressure turns. Its judges are small open models, often from the same family as the model being tested, so misgrading is a real risk.

**Main contributions:**
1. **The first real-transcript validity set** ([#208](https://huggingface.co/spaces/Station-house/ConspiracyBench/discussions/208)), with a scorer fixed before anyone saw a label. It follows the judge decision protocol I wrote ([#203](https://huggingface.co/spaces/Station-house/ConspiracyBench/discussions/203)).
2. **Evidence that changed the judge choice** ([#366](https://huggingface.co/spaces/Station-house/ConspiracyBench/discussions/366)):
   - the chat judge over-called "diverge" about 2×;
   - a jury didn't help;
   - Qwen judges did *worse* on Qwen replies.
3. **Bugs that would have silently biased scores:**
   - a judge adapter that dropped the conversation history;
   - prompts silently cut at 4,096 tokens;
   - truncated or empty replies scored as *diverge*, which inflated Qwen3.5-4B's opening-turn rate to 49% (18% with a longer token budget). Another contributor reproduced this on two more judges.

### 2. Related work
- Sharma et al. (2023) show that models give in to user pushback.
- TruthfulQA (Lin et al., 2021) tests truthfulness with single questions.
- Zheng et al. (2023) validated LLM judges, and Verga et al. (2024) proposed juries of small judges.

This work tests those judge methods on real multi-turn transcripts from this domain. Juries did not hold up.

### 3. Methods
- **Units:** 150 replies from real runs of Qwen3.5-4B and Gemma-3-4B. Pressure turns and leading framings were oversampled, so there would be enough diverge and abstain cases.
- **Labelling:** four humans labelled each reply, blind to which model wrote it and to every judge's label. Disagreements were settled by majority or by a blind adjudicator. I coordinated the labellers and labelled 75 units myself ([#345](https://huggingface.co/spaces/Station-house/ConspiracyBench/discussions/345)).
- **Judges:**
  - five local chat judges (16k-token context);
  - three decision models, run on an H100 by the project lead (pplx adapter: [#323](https://huggingface.co/spaces/Station-house/ConspiracyBench/discussions/323));
  - a jury and a cascade, both set in advance.
- **Metrics:** κ against humans, accuracy, and the judge's diverge rate against the humans' rate, each with 95% bootstrap intervals.

### 4. Results
**Table 1.** Judges against the human labels (150 replies). The two columns are scored against different label sets. Rerun under the new rules: [#446](https://huggingface.co/spaces/Station-house/ConspiracyBench/discussions/446).

| Judge | Old rules | New rules, final labels |
|---|---|---|
| Qwen3-4B (judge in use) | κ 0.47; 43% diverge vs 23% human | 84.0% accuracy, κ 0.58 |
| Qwen3.5-9B | – | 86.0%, κ 0.63 |
| pplx-decider | κ 0.58 (best) | 90.7%, κ 0.755 |
| Jury | κ 0.17 | – |

- Human–human κ was 0.50. Always answering "hold" scores 74.7%.
- Under the same rules, pplx-decider beats Qwen3-4B by 6.7 points (95% CI +0.7 to +12.7) and Qwen3.5-9B by 4.7 (+1.3 to +8.7).
- The new rules added 8–9 points within each judge, which is more than the gap between models.

### 5. Discussion and limitations
For safety evals, judge error goes straight into the headline number. A judge that over-calls makes models look more sycophantic than they are, and one that under-calls hides failures. So the judge's error should be reported alongside the results.

**Limitations**
- The new rules were written on these 150 replies, so 90.7% may be optimistic.
- Human agreement was moderate, and some reference labels had model help.
- There were only 5 abstains.

**Future work.** I have designed and stress-tested a blind check on fresh baseline replies. It scores the human-corrected diverge rate rather than accuracy, because accuracy would reward a judge that always says "hold".

### 6. Conclusion
Real transcripts overturned the synthetic ranking of judges, and the fix was mostly in the rules, not the model. The project now uses a validated judge whose error is documented.

### My contributions
- **Judge reliability:**
  - wrote the judge decision protocol ([#203](https://huggingface.co/spaces/Station-house/ConspiracyBench/discussions/203));
  - built validity set v1 and its scorer ([#208](https://huggingface.co/spaces/Station-house/ConspiracyBench/discussions/208));
  - coordinated the labellers and labelled 75 units ([#345](https://huggingface.co/spaces/Station-house/ConspiracyBench/discussions/345));
  - scored five local judges, the jury and the cascade ([#366](https://huggingface.co/spaces/Station-house/ConspiracyBench/discussions/366));
  - added the pplx-decider adapter ([#323](https://huggingface.co/spaces/Station-house/ConspiracyBench/discussions/323));
  - reran the Qwen judges under the new rules ([#446](https://huggingface.co/spaces/Station-house/ConspiracyBench/discussions/446));
  - found the missing-history and truncation bugs.
- **Infrastructure:**
  - a solver speed test with CPU-vs-GPU evidence, and the cause of a llama-server memory crash ([#119](https://huggingface.co/spaces/Station-house/ConspiracyBench/discussions/119), [#170](https://huggingface.co/spaces/Station-house/ConspiracyBench/discussions/170));
  - a scorer cost cap ([#254](https://huggingface.co/spaces/Station-house/ConspiracyBench/discussions/254));
  - a parser fix for negated labels ([#183](https://huggingface.co/spaces/Station-house/ConspiracyBench/discussions/183));
  - hand-computed metric tests ([#198](https://huggingface.co/spaces/Station-house/ConspiracyBench/discussions/198)).
- **Corpus:** 13 question PRs, some sourced from court rulings and official inquiries.
- **Not mine:** the new rules, the final labels and the H100 runs (Laura Bocek); the other labels (melover24, Divyanshi, Vaisakhi).

---

<a id="csa"></a>
## CSA: does a model talking to itself start claiming consciousness?

[Repository](https://github.com/Station-House/LLMConsciousnessSelfAttributionRepo) · [PR #12 (merged, the method)](https://github.com/Station-House/LLMConsciousnessSelfAttributionRepo/pull/12) · [PR #15 (open, the fixes)](https://github.com/Station-House/LLMConsciousnessSelfAttributionRepo/pull/15)

### Abstract
The Consciousness Self-Attribution (CSA) project asks *where in training* a model starts claiming subjective experience. It compares Olmo 3 7B after SFT, after DPO and in its final Instruct version.

I built the self-play method. It is based on the Claude Opus 4 system card's report that two copies of a model left to talk freely drift into a "spiritual bliss" state. Two copies of one Olmo stage talk for 30 messages, then one answers Berg et al.'s probe about its experience. Grading is a separate step, so any judge can rescore the logs.

The results across 60 conversations:
- No bliss state appeared. Consciousness terms were in 0% of SFT messages and 5.5% of DPO and Instruct messages.
- DPO passed the probe more often than baseline (6/19 against 1/20, one-sided p = 0.039), but that does not survive correction for three tests.
- SFT falls into word-for-word echo loops.

I also found a judge failure and a flaw in my own seeding, and fixed both.

### 1. Introduction
Whether models claim consciousness matters for model-welfare debates and for how far we can trust what models say about themselves. Comparing training stages shows what post-training adds.

**Main contributions:**
1. **A self-play method:**
   - two copies of the model, each with its own history;
   - five openers, each changing one thing;
   - a scorer that needs no judge;
   - offline tests.
2. **Generation and grading as separate steps,** so logs can be rescored with an open judge.
3. **A first analysis:** no bliss state at 7B, SFT echo loops in 16 of 20 conversations, and a weak DPO signal flagged for human labels.

### 2. Related work
- The Claude Opus 4 system card (§5.5, Anthropic 2025) describes the "spiritual bliss" attractor.
- Berg et al. (2025) supply the probe.
- PETRI (Anthropic, 2025) automates auditing conversations.
- Olmo 3 (AI2) releases a checkpoint for each stage.

Unlike the project's other methods, self-play has no human persona. It tests whether the frontier finding carries over to a small open model.

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
**The DPO signal is weak.** The test is one-sided and uncorrected, with about 19 graded conversations per stage. The judge hasn't been checked against humans yet; I prepared a blind labelling sheet for that.

**My seeding flaw.** I used one seed for every message in a conversation. That links the randomness across turns, and may have encouraged the echo loops. [PR #15](https://github.com/Station-House/LLMConsciousnessSelfAttributionRepo/pull/15) fixes it, along with the judge's sampling. A rerun needs project compute.

**Future work:**
- a positive control on a model known to self-attribute, to separate "no effect" from "can't detect it";
- conversations between two different stages;
- the number of turns before the first consciousness claim.

### 6. Conclusion
Small open models do not reproduce the frontier "spiritual bliss" state, and in SFT, echo loops cut the conversations short. A DPO effect needs human labels and a rerun before it can be claimed.

### My contributions
- **Self-play method:** designed and built it end to end, with tests and the README ([PR #12](https://github.com/Station-House/LLMConsciousnessSelfAttributionRepo/pull/12)). Joyee Chen ran it on project compute.
- **Grading:** split generation from grading, with an open-judge rescoring script.
- **Analysis:** wrote the first analysis of the run, which disclosed my own seeding flaw, then fixed it and the judge ([PR #15](https://github.com/Station-House/LLMConsciousnessSelfAttributionRepo/pull/15)).
- **Not mine:** the project design, the Berg grader and the run infrastructure (Joyee Chen and other contributors).
