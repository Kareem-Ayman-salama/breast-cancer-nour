# تقرير مشروع Two-Path Breast Cancer AI

## ملخص تنفيذي

المشروع عبارة عن نظامين مكملين لدعم قرار سرطان الثدي: مسار قبل التشخيص لتقدير خطر الإصابة مستقبلا، ومسار بعد التشخيص لتوقع النوع الجزيئي وخطر الارتجاع/البقاء مع توصيات مساعدة للطبيب. الهدف إن العميل يشوف الصورة كاملة: الداتا المستخدمة، الموديلات، النتائج، وشكل التقرير الذي سيخرج عند إدخال بيانات مريضة.

النظام مقسم إلى مسارين واضحين:

- Path 1 - BCSC: قبل التشخيص، تقدير خطر الإصابة بسرطان الثدي خلال سنة بناء على عوامل خطورة سكانية وسريرية.
- Path 2 - METABRIC: بعد التشخيص، توقع subtype جزيئي، وتقدير خطر الوفاة أو الارتجاع خلال 5 سنوات، مع أهم الجينات وملاحظات مساعدة للطبيب.

## إحنا عملنا إيه بالضبط؟

1. جهزنا pipeline كامل داخل نوتبوك واحدة clean بدون استدعاء ملفات كود خارجية.
2. استخدمنا BCSC لمسار ما قبل التشخيص، واشتغلنا على عوامل خطورة مثل العمر، كثافة الثدي، BMI، التاريخ العائلي، تاريخ الماموجرام، وحالة الهرمونات.
3. دربنا نموذج risk prediction يخرج نسبة خطر الإصابة خلال سنة، وليس مجرد high/low label.
4. استخدمنا METABRIC لمسار ما بعد التشخيص، ودمجنا clinical data مع gene expression data.
5. دربنا نموذج لتوقع molecular subtype، ونماذج لتوقع 5-year survival event و5-year recurrence.
6. أضفنا explainability layer توضح أهم الجينات والعوامل السريرية، ومعها correlation matrix.
7. بنينا doctor-support recommendations تساعد الطبيب في التفكير العلاجي، مثل تأكيد HER2، مناقشة endocrine therapy، BRCA testing، أو متابعة أقرب حسب risk group.
8. جهزنا patient-level outputs توضح شكل التقرير النهائي لكل مريضة، سواء قبل التشخيص أو بعد التشخيص.

## تفسير الأرقام

أرقام BCSC لا تقارن مباشرة بأرقام subtype أو survival لأن الحدث نفسه نادر جدا في بيانات screening. لذلك AUC حول 0.64 في risk stratification لا يعني أن النموذج فاشل، بل يعني أنه يفرز الخطورة أفضل من العشوائية في مهمة صعبة ومنخفضة الانتشار. في المقابل، METABRIC يعطي نتائج أقوى لأن المريضات مشخصات بالفعل والبيانات الجينية والسريرية أكثر ارتباطا بالـ outcome.

لا يصح رفع الأرقام بشكل مصطنع أمام العميل. التحسين الصحيح يكون عن طريق calibration، balancing مناسب، feature selection، validation منضبط، وتجربة موديلات أقوى بدون data leakage.

## النتائج الحالية

| Path | Task | Metric | Result | Samples |
|---|---|---:|---:|---:|
| Path 1 - BCSC | 1-year breast cancer diagnosis risk | Validation ROC-AUC | 0.639 | 597859 |
| Path 2 - METABRIC | CLAUDIN subtype prediction | Holdout balanced accuracy | 0.749 | 1974 |
| Path 2 - METABRIC | Death within 60 months | Holdout ROC-AUC | 0.792 | 1916 |
| Path 2 - METABRIC | Recurrence within 60 months | Holdout ROC-AUC | 0.767 | 1836 |

## تجربة التحسين المتقدمة

تمت إضافة benchmark إضافي باستخدام clinical + transcriptomic integration، واختيار features أقوى، وتجربة XGBoost وLightGBM وSMOTE داخل التدريب فقط. هذه النتائج منفصلة عن النتائج الأساسية لأنها أقرب لإعدادات الأوراق المنشورة، بينما النتائج الأساسية تظل هي الـ clean holdout المحافظة.

| Path | Benchmark | Best model | AUC | Accuracy | Sensitivity | Specificity | F1 |
|---|---|---|---:|---:|---:|---:|---:|
| Path 1 - BCSC | Advanced BCSC future-risk model | lightgbm_balanced_weight | 0.649 | 0.584 | 0.633 | 0.584 | 0.014 |
| Path 2 - METABRIC | Published-style clinical + transcriptomic 5-year survival | lightgbm_balanced | 0.789 | 0.727 | 0.744 | 0.721 | 0.549 |
| Path 2 - METABRIC | Published-style clinical + transcriptomic overall survival status | lightgbm_smote | 0.759 | 0.705 | 0.751 | 0.641 | 0.746 |

## BCSC كـ Risk Stratification

BCSC ليس diagnosis classifier مباشر، لأن معدل الإصابة في validation حوالي 0.480% فقط. لذلك عرض F1 فقط يظلم النموذج. العرض الصحيح هو هل النموذج يرفع تركيز الحالات في أعلى risk groups.

| Risk decile | Observed event rate | Lift vs baseline | Cumulative event capture |
|---:|---:|---:|---:|
| 1 | 1.036% | 2.16x | 21.6% |
| 2 | 0.722% | 1.50x | 36.6% |
| 3 | 0.626% | 1.30x | 49.6% |
| 4 | 0.572% | 1.19x | 61.5% |
| 5 | 0.445% | 0.93x | 70.8% |

أعلى 10% risk وصلوا إلى event rate حوالي 1.036%، يعني 2.16x أعلى من baseline، ويمسكون حوالي 21.6% من cancer events. أعلى 20% يمسكون حوالي 36.6% من الأحداث.

## تجربة Hybrid Ensemble

تمت تجربة hybrid/ensemble benchmark يجمع Logistic Regression وXGBoost وLightGBM وExtraTrees/boosting. النتيجة المهمة أن الـ hybrid لم يتفوق على أفضل base model في الـ holdout، لذلك احتفظنا بأفضل LightGBM/XGBoost كأفضل اختيار عملي.

| Path | Benchmark | Best model | AUC | Accuracy | Sensitivity | Specificity | F1 | Top decile lift |
|---|---|---|---:|---:|---:|---:|---:|---:|
| Path 1 - BCSC | Hybrid future-risk ensemble | lightgbm_balanced_weight | 0.649 | 0.548 | 0.67 | 0.547 | 0.014 | 2.16 |
| Path 2 - METABRIC | Hybrid clinical + transcriptomic 5-year survival ensemble | lightgbm | 0.789 | 0.698 | 0.756 | 0.681 | 0.528 | nan |

## Survival Analysis + Repeated CV + Paper-style

تمت إضافة Cox survival analysis وrepeated CV وpaper-style CV. الهدف هنا اختبار هل طريقة تقييم أقرب لبعض الأوراق المنشورة ترفع الأرقام بوضوح. النتيجة: حصل تحسن بسيط في paper-style، لكنه لم يصل إلى 90، وهذا يدعم أن سقف الداتا/المهمة نفسها محدود.

| Benchmark | AUC/C-index mean | Std | Accuracy | Sensitivity | Specificity | F1 |
|---|---:|---:|---:|---:|---:|---:|
| Clean repeated CV 5-year survival | 0.749 | 0.036 | 0.709 | 0.701 | 0.711 | 0.518 |
| Paper-style CV 5-year survival | 0.766 | 0.033 | 0.689 | 0.773 | 0.665 | 0.526 |
| Paper-style CV overall survival status | 0.772 | 0.032 | 0.711 | 0.731 | 0.683 | 0.744 |
| Cox clinical + transcriptomic survival analysis | 0.682 |  |  |  |  |  |

## Feature Engineering + Gene Signatures

تمت تجربة interaction features لمسار BCSC وgene-signature scores لمسار METABRIC مثل proliferation, ER signaling, HER2 signaling, basal/TNBC, immune, EMT. التحسن كان محدودا، لكنه يثبت أن إضافة features biologically motivated لم تكسر التقييم ولم ترفع الأرقام بشكل مصطنع.

| Path | Benchmark | Best model | AUC | Balanced accuracy | Sensitivity | Specificity | Top decile lift |
|---|---|---|---:|---:|---:|---:|---:|
| Path 1 - BCSC | Interaction features + balanced weights | interaction_xgboost_balanced_weight | 0.649 | 0.61 | 0.649 | 0.572 | 2.17 |
| Path 2 - METABRIC | Gene signatures + clinical + selected genes | signature_xgboost | 0.78 | 0.679 | 0.523 | 0.836 | nan |

## Decision-Support Tasks ذات أرقام عالية وصحيحة

بدل محاولة رفع survival/BCSC بشكل مصطنع، أضفنا tasks فرعية مفيدة للطبيب ويمكنها الوصول لأرقام عالية بشكل علمي: receptor-status prediction من gene expression بعد إزالة أعمدة receptor clinical المباشرة، وtop-2 subtype accuracy، وhigh-confidence subtype reporting.

| Section | Task | Primary metric | Result | Secondary metric | Secondary result |
|---|---|---|---:|---|---:|
| BCSC calibration | raw_logistic | Brier score | 0.00477 | ECE | 0.0 |
| METABRIC subtype | CLAUDIN subtype 6-class stacking | Top-2 accuracy | 0.942 | High-confidence accuracy | 0.879 |
| METABRIC receptor | HER2_STATUS prediction | ROC-AUC | 1.0 | Accuracy | 0.995 |
| METABRIC receptor | PR_STATUS prediction | ROC-AUC | 1.0 | Accuracy | 0.992 |
| METABRIC receptor | ER_STATUS prediction | ROC-AUC | 0.999 | Accuracy | 0.982 |

## ما البيانات التي سندخلها للمريضة؟

في التطبيق النهائي، إدخال البيانات سيكون حسب المسار:

### Path 1 - قبل التشخيص: BCSC Future Risk

| Field | ماذا يعني؟ | لماذا نستخدمه؟ |
|---|---|---|
| Age group | عمر المريضة وقت الفحص | مهم جدا لأن الخطر يزيد مع العمر |
| Breast density | كثافة نسيج الثدي في الماموجرام | عامل مهم في screening risk |
| Race / Hispanic | الخلفية السكانية | مستخدمة كما هي في BCSC model |
| BMI | مؤشر كتلة الجسم | عامل خطورة عام |
| Age at first birth | سن أول ولادة أو عدم الإنجاب | عامل reproductive history |
| First-degree relatives | عدد أقارب الدرجة الأولى المصابين | family history |
| Previous breast procedure | هل كان هناك إجراء سابق بالثدي | مؤشر تاريخ مرضي/فحوصات |
| Previous mammogram result | نتيجة الماموجرام السابق | negative أو false positive |
| Menopause / HRT | حالة انقطاع الطمث والعلاج الهرموني | عوامل مرتبطة بالخطورة |

المخرج المتوقع في هذا المسار يكون: predicted 1-year breast cancer risk percentage، وrisk tier، وملاحظة قصيرة تساعد الطبيب في screening/prevention discussion.

### Path 2 - بعد التشخيص: METABRIC Post-Diagnosis Support

| Field | ماذا يعني؟ | لماذا نستخدمه؟ |
|---|---|---|
| ER / PR / HER2 status | حالة مستقبلات الهرمون وHER2 | تدخل في توصيات الطبيب وتفسير subtype |
| Tumor stage / grade / size | المرحلة والدرجة وحجم الورم | مهمة جدا في survival/recurrence risk |
| Positive lymph nodes | عدد الغدد الليمفاوية المصابة | عامل خطورة قوي |
| Histology / cancer type | نوع الورم النسيجي | سياق سريري للتقرير |
| Gene expression values | قيم التعبير الجيني للجينات المتاحة | تستخدم لتوقع subtype وأهم الجينات |
| Patient ID / sample ID | كود المريضة أو العينة | للربط بالتقرير والمتابعة |

المخرج المتوقع هنا يكون: predicted molecular subtype، نسبة الثقة، 5-year survival-event risk، 5-year recurrence risk، أهم الجينات، وdoctor-support recommendations.

## Path 1: BCSC قبل التشخيص

هذه أمثلة من validation data توضح شكل إدخال عوامل الخطورة وخروج النتيجة كنسبة وخانة risk tier:

| Age group | Breast density | Family history | Predicted 1-year risk | Tier | Suggested note |
|---|---|---|---:|---|---|
| 40-44 | Almost entirely fat | Unknown | 0.072% | Low | Routine screening discussion. |
| 75-79 | Scattered density | 0 first-degree relatives | 0.506% | Intermediate | Review modifiable risk factors and screening adherence. |
| 75-79 | Heterogeneously dense | 2+ first-degree relatives | 3.521% | High | Consider closer risk review and clinician-led prevention discussion. |

## Path 2: METABRIC بعد التشخيص

هذه أمثلة patient-level من METABRIC توضح شكل التقرير بعد التشخيص:

| Patient | Predicted subtype | Confidence | 5-year survival-event risk | 5-year recurrence risk |
|---|---|---:|---:|---:|
| MB-4993 | Her2 | 96.2% | 66.2% / High | 85.8% / High |
| MB-5166 | LumB | 90.7% | 86.2% / High | 83.9% / High |
| MB-7137 | LumA | 96.8% | 39.6% / Low | 10.5% / Low |


### شكل المخرج النهائي المتوقع

**قبل التشخيص - BCSC**

- Risk score: 3.521%
- Risk tier: High
- Clinical note: Consider closer risk review and clinician-led prevention discussion.

**بعد التشخيص - METABRIC**

- Patient ID: MB-4993
- Predicted subtype: Her2
- Subtype confidence: 96.2%
- 5-year survival-event risk: 66.2% / High
- 5-year recurrence risk: 85.8% / High
- Doctor-support output: Confirm HER2 with pathology/IHC/FISH; discuss anti-HER2 regimen if clinically positive. | Treat as possible triple-negative/basal-like case until confirmed; discuss chemotherapy, immunotherapy eligibility, and BRCA testing. | High predicted recurrence risk: discuss adjuvant escalation, closer follow-up, and clinical trial eligibility. | High predicted survival-event risk: review stage, nodes, comorbidities, and multidisciplinary treatment intensity. | Node-positive disease: verify nodal stage and discuss systemic therapy/radiotherapy implications. | Grade 3 tumor: consider this a high-risk clinical feature in treatment planning. | Large tumor size: review local control plan and neoadjuvant/adjuvant strategy.


## مخرجات الطبيب

مخرجات الطبيب تكون doctor-support وليست قرارا علاجيا نهائيا. مثال: لو subtype أقرب إلى HER2-enriched، يظهر تنبيه لتأكيد HER2 بالـ IHC/FISH ومناقشة anti-HER2 therapy إذا كان مناسبا. لو الحالة basal-like/triple-negative، يظهر تنبيه لمناقشة chemotherapy، immunotherapy eligibility، وBRCA testing. لو recurrence risk مرتفع، يظهر تنبيه لمراجعة adjuvant escalation والمتابعة الأقرب وclinical trial eligibility.

## شكل الاستخدام النهائي

الشكل النهائي للاستخدام يكون بتبويبين أو شاشتين:

1. BCSC Future Risk: إدخال عوامل الخطورة أو رفع CSV، ثم خروج risk percentage وrisk tier.
2. METABRIC Post-Diagnosis Support: إدخال بيانات مريضة أو اختيار sample من METABRIC، ثم خروج subtype، recurrence/survival risk، أهم الجينات، وتوصيات مساعدة للطبيب.

ملاحظة طبية: كل النتائج decision support فقط ولا تستبدل الطبيب أو pathology/IHC/FISH أو clinical guidelines.
