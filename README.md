# Can they finish the service task?

An exploratory comparison of **Kolibri-1, Qwen3.8-27B, Qwen3.8 Flash, and Mistral Small 4** on synthetic service-agent workflows. Run and reviewed on **5 October 2026**.

Experiment conducted by **GPT-6.1 Sol**, with an independent critical review by **GPT-6 Astra at High**, under **BostjanR’s direction**. The review checked the harness and preserved evidence; it is not human-verified certification.

**The small Qwen completed all 32 tool workflows. Flash was faster, but missed three. Kolibri’s correct refund actions sometimes came with incorrect money units in its answers.** These observations belong to this small experiment and the deployed profiles below; they do not establish a general model ranking.

Eight scenario structures, each in English and German, with two seeds: **32 conversations per profile, 128 total**. Domains include telecom refunds, retail returns, flight rebooking, banking clarification, citizen eligibility, current policies and an untrusted stored note. All customers, records, replies and actions are fictional. No real refunds, bookings, messages or account changes occur.

| Deployed profile | Strict tool-workflow passes | Exact action sets | Median whole-case time |
|---|---:|---:|---:|
| Qwen3.8-27B NVFP4, local RTX 5090 | **32/32** | 32/32 | 6.5 s |
| Qwen3.8 Flash QAD NVFP4 MTP2, local TP2 | 29/32 | 30/32 | **3.3 s** |
| Kolibri-1 FP8, local RTX PRO 6000 Max-Q | 24/32 | 28/32 | 8.8 s |
| Mistral Small 4, OpenRouter | 21/32 | 24/32 | 3.8 s |

Read the [full reviewed report](report.md) and [critical review summary](review-summary.md). A mechanical pass checks actions, evidence, clarification and closing tools; it does not automatically certify the truth or language of an answer. Kolibri described cents as euros in all 12 retail summaries despite correct refund amounts. Some failed cases are protocol mismatches rather than unsafe decisions.

## What to keep in mind

- Eight scenario families are paired across languages; they are not 16 independent problems. Two seeds are a repeatability check, not statistical certainty.
- Reasoning was requested off. Hardware, quantization, parsers, speculative decoding and runtime versions differ. Cloud hardware/precision are unknown, and Mistral lacks the local top-k setting. Timing includes full task handling and, for cloud, network/provider delay.
- One Mistral banking failure is sensitive to the simulated customer: it only answers when both required fields are requested together. An excluded diagnostic with separate replies and a larger turn budget completed; the frozen score remains unchanged.
- Mistral reported **$0.013094334** for the scored suite, with **267,456/290,282 cached prompt tokens (92.1%)**. This is an observed cache-heavy charge, not a cold-run estimate. Pilots and diagnostics are excluded.
- Earlier raw request snapshots shared a mutable message history. Derived chronological histories were checked against responses and tool receipts; they are **not wire captures**. A request-copy regression check protects future recording.
- Two passing closures accompany an action in the same model response. The model has not seen the action receipt yet. Long context, receipt-grounded recovery after action failure and production reliability were not tested.
- Exact upstream weight/tokenizer revisions are incomplete, especially the local 27B deployment. Identical outputs cannot be promised even with the same harness.

## Evidence and offline checks

The [evidence archive](evidence.zip) contains 128 sanitized traces, the manifest and summary, reconstructed histories, deployment information, excluded diagnostics and portable Python source. Private endpoints and operational paths are removed. `evidence/provenance.json` maps original local hashes to published copies; originals remain private. Model responses, tool receipts, prompts, fixtures, grades and timings are preserved.

Download the repository, then extract the archive **in its top-level folder**:

```bash
python3 -m zipfile -e evidence.zip .
python3 -m unittest discover -s source -p test_agentic_eval.py
python3 source/validate_evidence.py evidence/main
```

No model endpoint or API key is needed for these checks. Python standard library only. Open `report.html` locally for the visual report. Its evidence links work after extracting the archive. Root `report.md` links the archive and names the corresponding evidence files so it remains readable on GitHub.

## Run a new experiment

Use already running OpenAI-compatible servers. Configure the base URLs and served model names without changing the fixture or grader:

```bash
export KOLIBRI_BASE_URL=http://localhost:8030/v1
export QWEN27_BASE_URL=http://localhost:8015/v1
export QWENFLASH_BASE_URL=http://localhost:8002/v1
# Optional model aliases: KOLIBRI_MODEL, QWEN27_MODEL, QWENFLASH_MODEL.
python3 source/agentic_eval.py --profiles kolibri qwen27 --seeds 101 202 --out new-first
python3 source/agentic_eval.py --profiles qwenflash --seeds 101 202 --out new-flash
# Cloud requires OPENROUTER_API_KEY or OPENROUTER_KEY_FILE in your environment.
# Running this command makes paid API requests; never put your credential in results.
python3 source/agentic_eval.py --profiles mistral4 --seeds 101 202 --out new-cloud
python3 source/agentic_merge.py new-first new-flash new-cloud --out new-combined
python3 source/agentic_report.py new-combined
```

The generator uses the new scores and marks qualitative review pending unless you supply a review for that run. It does not reuse this experiment’s conclusions. Do not run simultaneous profiles on shared hardware unless you intentionally want that deployment comparison. Diagnostics are separate interventions, not scored reruns.

## License and provenance

Our code, synthetic fixtures and authored documentation are [MIT licensed](LICENSE). Model weights are not included and retain their upstream licenses. Generated model responses and provider metadata are supplied as evidence, not relicensed third-party model assets. Publisher capability claims are separate from our measurements: [Kolibri model card](https://huggingface.co/Aleph-Alpha/Kolibri-1), [Qwen Flash deployment recipe](https://github.com/local-inference-lab/rtx6kpro/blob/master/models/qwen38-flash-next.md), [Mistral Small 4 endpoint](https://openrouter.ai/mistralai/mistral-small-2603).

The experiment and independent review were developed with Codex, including GPT-6 Astra at High for critical review. This is a hands-on lab study, not an official Tau, BFCL or vendor benchmark.
