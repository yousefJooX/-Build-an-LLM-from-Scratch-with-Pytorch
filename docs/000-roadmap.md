---
title: Jabarti LLM — Roadmap & Timeline
tags: [llm-from-scratch, roadmap, jabarti]
created: 2026-10-25
updated: 2026-10-25
---

# 🗺️ Jabarti LLM — Roadmap & Timeline

> **الهدف:** نخلص الكورس كله (7 سكاشن، ~16.5 ساعة فيديو) ونطلع بـ **GPT ثنائي اللغة شغال، متدرب على GPU، ومنشور على Hugging Face** — مش مجرد notebooks.
> **الإيقاع:** ~1.5 ساعة/يوم، 5–6 أيام في الأسبوع (الفيديو × 3 = فرجة + كتابة كود + تجارب).
> **المدة:** 26 أكتوبر → 6 يناير ≈ **10.5 أسبوع** (فيهم أسبوع buffer). التاسكات اليومية على StudyHub.

---

## 📅 Timeline

| # | Section | Branch | فيديو | الفترة | Doc |
|---|---------|--------|-------|--------|-----|
| 1 | Basics | `section-01-basics` | 2h | 🔁 مراجعة 26 → 28 أكتوبر | [[001-section-01-basics]] |
| 2 | Bilingual Tokenizer | `section-02-tokenizer` | 1h49 | 29 أكتوبر → 4 نوفمبر | [[002-section-02-tokenizer]] |
| 3 | PyTorch Basics | `section-03-pytorch` | 1h32 | 5 → 10 نوفمبر | [[003-section-03-pytorch]] |
| 4 | GPT Model from Scratch | `section-04-gpt-model` | 5h04 | 11 نوفمبر → 1 ديسمبر (3 أسابيع) | [[004-section-04-gpt-model]] |
| 5 | Data Loading & Generation | `section-05-data-generation` | 2h18 | 2 → 10 ديسمبر | [[005-section-05-data-generation]] |
| 6 | Pre-Training | `section-06-pretraining` | 2h29 | 11 → 23 ديسمبر | [[006-section-06-pretraining]] |
| 7 | LoRA Finetuning | `section-07-lora` | 1h17 | 24 → 30 ديسمبر | [[007-section-07-lora]] |
| ★ | Capstone & Release | `release-v1` | — | 31 ديسمبر → 6 يناير | [[008-capstone]] |

> ⚠️ سكشن 4 هو الأتقل (5 ساعات فيديو + أصعب مفاهيم) — اديله 3 أسابيع ومتستعجلش. سكشن 6 فيه training run حقيقي، فسيب وقت للـ GPU يشتغل.
> لو امتحانات أو مشروع التخرج ضغطوا، أسبوع الـ Capstone هو الـ buffer.

---

## 🌿 Git Workflow (لكل سكشن)

```bash
git checkout main && git pull
git checkout -b section-0N-<name>
# ... شغل + commits صغيرة ...
git push -u origin section-0N-<name>
# افتح PR على GitHub → Merge into main
git checkout main && git pull
git tag v0.N-section-0N && git push --tags
```

**Commit style:** `feat(s2): train BPE tokenizer` · `docs(s2): add 002 note` · `test(s4): attention shape tests`

---

## 📁 الـ Structure المستهدف

```text
.
├── docs/                 # 000 roadmap + نوت لكل سكشن (001..008)
├── sections/             # notebooks + Obsidian notes لكل سكشن
│   ├── 01-basics/
│   ├── 02-tokenizer/
│   └── ...
├── jabarti/              # الكود الحقيقي كـ Python package (بيكبر سكشن ورا سكشن)
│   ├── tokenizer.py      # S2
│   ├── model.py          # S4 (embeddings, attention, block, GPT)
│   ├── data.py           # S5
│   ├── generate.py       # S5
│   └── train.py          # S6
├── tests/                # pytest — shapes, causal mask, KV cache = no-cache output
├── configs/              # YAML configs للـ training
└── README.md
```

> 💡 الفكرة: الـ notebooks للتعلّم، والـ `jabarti/` package هو المنتج. كل سكشن بيضيف module.

---

## 🚀 الـ Extras (اللي هتخلّي المشروع يبان فوق الكورس)

| Section | Extra |
|---------|-------|
| S2 | قيس **fertility** (tokens/word) عربي vs إنجليزي، وقارن بالـ GPT-2 / tiktoken tokenizer |
| S3 | ابني autograd صغير (micrograd-style) عشان تفهم `backward()` |
| S4 | `tests/` للـ shapes + الـ causal mask · حساب الـ parameter count يدوي · جرّب `F.scaled_dot_product_attention` · (bonus) RoPE |
| S5 | Benchmark للـ KV cache (tokens/sec مع وبدون) · repetition penalty |
| S6 | Mixed precision (bf16/fp16) + gradient accumulation على الـ RTX 4050 (6GB) · قيس perplexity · `torch.compile` |
| S7 | اكتب LoRA layer بإيدك الأول، وبعدين قارن بـ `peft` |
| ★ | ارفع الموديل على **HF Hub** + **Gradio demo** على HF Spaces + README فيه loss curves و samples |

---

## ✅ Definition of Done لكل سكشن

- [ ] اتفرجت على الفيديوهات وكتبت Obsidian notes
- [ ] الكود اتكتب بإيدي (مش copy-paste) واشتغل
- [ ] عملت تجربة واحدة على الأقل برا الكورس (Extra)
- [ ] كتبت `docs/00N-*.md`
- [ ] Push + PR + Merge + Tag
- [ ] حدثت الـ README (Learning Progress)
