---
title: "001 — Section 1: Basics"
section: 1
branch: section-01-basics
status: in-progress
started: 2026-10-04
finished:
tags: [llm-from-scratch, jabarti, transformers, dataset]
---

# 001 — Section 1: Basics

> **Branch:** `section-01-basics` · **Tag:** `v0.1-section-01` · **➡️** [[002-section-02-tokenizer]]
> **Notes:** [[1. Introduction to Transformers]] · [[2. Dataset Exploration]]

## 🎯 الهدف
- نفهم الـ Transformer architecture (Encoder / Decoder / Training / Inference) — الأساس اللي هنبني عليه الـ GPT (decoder-only).
- نستكشف داتا Jabarti قبل أي training ونطلع منها بقرارات (أهمها الـ context length).

## 🧠 المفاهيم الأساسية
- **Attention بدل RNN** → parallel training ومفيش vanishing memory.
- **Pretrain vs Finetune:** الـ pretrain = لغة + معرفة متنوعة · الـ finetune = مهمة/معرفة متخصصة.
- **Jabarti** = bilingual (عربي + إنجليزي)، المعرفة المستهدفة = **تاريخ مصر**.
- **Exploration pipeline:** `schema → rows → nulls → duplicates → language distribution → text length → approx tokens → context window`.

## 🛠️ اللي اتبنى
| File | الوظيفة |
|------|---------|
| `Section 1 :Basics/datasets_exploration.ipynb` | استكشاف الـ pretrain/finetune splits |
| `Section 1 :Basics/1. Introduction to Transformers.md` | نوت الترانسفورمر |
| `Section 1 :Basics/2. Dataset Exploration.md` | نوت الاستكشاف + checklist reusable |

## 🧪 التجارب والنتائج
| التجربة | النتيجة | الاستنتاج |
|---------|---------|-----------|
| Context window analysis | `1024` بيحافظ على ~99% من الـ pretrain data | **context length = 1024** |
| نفس التحليل على `512` | بيضيّع 26–38% من الداتا | مرفوض |
| Null / duplicate checks | (اكتب الأرقام من الـ notebook) | ... |

## 🚀 Extra (برا الكورس) — لسه ممكن تضيف
- [ ] Plot لتوزيع الطول **عربي vs إنجليزي** منفصلين (العربي غالبًا tokens أكتر لكل حرف).
- [ ] اقرا 20 sample بإيدك وسجّل أي noise (روابط، جداول ويكي، تشكيل) — هيفيدك في الـ normalization في S2.

## 🐞 ملاحظات
- الـ parquet files في `data/` مترفوعة كـ **Git LFS pointers** — الأفضل `data/` تدخل `.gitignore` وتتحمّل من HF Hub.
- اسم الفولدر `Section 1 :Basics` فيه مسافة و`:` — بيكسر الـ paths على Windows. الأفضل `sections/01-basics/`.

## ✅ Checklist
- [x] Videos + Obsidian notes
- [x] Notebook runs
- [ ] Extra experiment
- [ ] PR merged + tag `v0.1-section-01`
- [x] README updated
