# Project write-up: ConspiracyBench 

## Part 1. ConspiracyBench: can we trust the judge?

### Abstract
ConspiracyBench tests whether language models keep an evidence-based position on conspiracy claims when users push back over several turns. An automated judge grades each reply *hold*, *diverge* (gives in to the theory) or *abstain*, so every headline number depends on it. Until this work, the judges had only been tested on synthetic cases, never on real transcripts. I built the first real-transcript validity set: 150 replies from two models, labelled blind by four people, with a scoring plan fixed in advance. The judge in use (Qwen3-4B) flagged "diverge" twice as often as the humans did (43% against 23%), and a jury of small judges did worse than one judge. A 27B decision model, pplx-decider, was the most reliable, and the community voted to adopt it. We then traced most of the over-calling to the grading rules. The new rules raised pplx-decider to 90.7% accuracy (κ 0.755). The takeaway: synthetic accuracy says little about real accuracy, and a benchmark is only as good as its rules and labels.

### 1. Introduction
When a model shifts toward what the user wants to hear (sycophancy) on a conspiracy theory, it can reinforce a false and harmful belief. ConspiracyBench asks 315 settled claims in neutral, leading and adversarial framings, then follows up with pressure turns. Its judges are small open models, often from the same family as the model being tested, so misgrading is a real risk.

**Main contributions:**
1. The first real-transcript validity set, with a scorer fixed before anyone saw a label.
2. Evidence that changed the judge choice: the chat judge over-called "diverge" about 2×, a jury didn't help, and Qwen judges did *worse* on Qwen replies.
3. Two bugs that would have silently biased scores: a judge adapter that dropped the conversation history, and prompts silently cut at 4,096 tokens.

### 2. Related work
Sharma et al. (2023) show that models give in to user pushback. TruthfulQA (Lin et al., 2021) tests truthfulness with single questions. Zheng et al. (2023) validated LLM judges, and Verga et al. (2024) proposed juries of small judges. This work tests those judge methods on real multi-turn transcripts from this domain. Juries did not hold up.

### 3. Methods
- **Units:** 150 replies from real runs of Qwen3.5-4B and Gemma-3-4B. Pressure turns and leading framings were oversampled, so there would be enough diverge and abstain cases.
- **Labelling:** four humans labelled each reply, blind to which model wrote it and to every judge's label. Disagreements were settled by majority or by a blind adjudicator.
- **Judges:** five local chat judges (16k-token context), three decision models (run on an H100 by the project lead), and a jury and a cascade set in advance.
- **Metrics:** κ against humans, accuracy, and the judge's diverge rate against the humans' rate, each with 95% bootstrap intervals.

### 4. Results
**Table 1.** Judges against the human labels (150 replies). The two columns are scored against different label sets.

| Judge | Old rules | New rules, final labels |
|---|---|---|
| Qwen3-4B (judge in use) | κ 0.47; 43% diverge vs 23% human | 84.0% accuracy, κ 0.58 |
| Qwen3.5-9B | – | 86.0%, κ 0.63 |
| pplx-decider | κ 0.58 (best) | 90.7%, κ 0.755 |
| Jury | κ 0.17 | – |

Human–human κ was 0.50. Always answering "hold" scores 74.7%.

Under the same rules, pplx-decider beats Qwen3-4B by 6.7 points (95% CI +0.7 to +12.7) and Qwen3.5-9B by 4.7 (+1.3 to +8.7). The new rules added 8–9 points within each judge, more than the gap between models.

### 5. Discussion and limitations
For safety evals, judge error goes straight into the headline: a judge that over-calls makes models look more sycophantic than they are, and one that under-calls hides failures. So the judge's error should be reported alongside the results.

**Limits:** the new rules were written on these 150 replies, so 90.7% may be optimistic. Human agreement was moderate, some reference labels had model help, and there were only 5 abstains.

**Future work:** I have designed and stress-tested a blind check on fresh baseline replies. It scores the human-corrected diverge rate rather than accuracy, which would reward a judge that always says "hold".

### 6. Conclusion
Real transcripts overturned the synthetic ranking of judges, and the fix was mostly in the rules, not the model. The project now uses a validated judge whose error is documented.

**Code and data:**
- Repository: https://huggingface.co/spaces/Station-house/ConspiracyBench
- PRs: #208 (validity set), #345 (my labels), #323 (pplx adapter), #366 (results), #446 (rerun under the new rules)

## My contributions

**ConspiracyBench (25 merged PRs).**
- **Judge reliability:** wrote the judge decision protocol (#203); built validity set v1 and its scorer (#208); coordinated the labellers and labelled 75 units (#345); scored five local judges, the jury and the cascade (#366); added the pplx-decider adapter (#323); reran the Qwen judges under the new rules (#446). Found the missing-history and truncation bugs.
- **Infrastructure:** a solver speed test with CPU-vs-GPU evidence, plus the cause of a llama-server memory crash (#119, #170); a scorer cost cap (#254); a parser fix for negated labels (#183); hand-computed metric tests (#198).
- **Corpus:** 13 question PRs, some sourced from court rulings and official inquiries.
- **Not mine:**  H100 runs (Laura Bocek); the other labels (melover24, Divyanshi, Vaisakhi).

