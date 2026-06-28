# Recommendation Knowledge Base

This file documents the clinical logic used by the Streamlit decision-support recommendations. The app is not a treatment guideline engine; it provides clinician-facing prompts that must be checked against pathology, IHC/FISH, local protocols, and current guidelines.

## Rules Used in the Dashboard

| Trigger | App recommendation | Scientific rationale | Source |
|---|---|---|---|
| ER_STATUS or PR_STATUS is Positive | Discuss endocrine therapy suitability; check menopausal status and contraindications. | Hormone-receptor-positive breast cancer is commonly managed with adjuvant endocrine therapy; extended AI therapy may be offered in node-positive disease. | ASCO adjuvant endocrine therapy guideline update: https://ascopubs.org/doi/10.1200/JCO.18.01160 |
| HER2_STATUS is Positive or predicted subtype is Her2 | Confirm HER2 with pathology/IHC/FISH; discuss anti-HER2 regimen if clinically positive. | HER2-directed treatment depends on accurate HER2 testing; ASCO/CAP guidance emphasizes HER2 testing interpretation and IHC/ISH workflow. | CAP HER2 testing guideline update: https://www.cap.org/protocols-and-guidelines/cap-guidelines/current-cap-guidelines/recommendations-for-human-epidermal-growth-factor-2-testing-in-breast-cancer |
| Basal-like/TNBC pattern or ER-/PR-/HER2- | Discuss chemotherapy, immunotherapy eligibility, and BRCA/genetic testing where appropriate. | TNBC treatment commonly includes surgery/chemotherapy; immunotherapy such as pembrolizumab may be used with chemotherapy in selected settings. | NCI TNBC treatment overview: https://www.cancer.gov/types/breast/treatment/triple-negative-breast-cancer |
| Recurrence risk is High | Review adjuvant escalation, closer follow-up, and clinical trial eligibility. | High predicted risk is not itself a guideline indication, but it is a prompt for multidisciplinary review and closer risk discussion. | Model-derived decision-support rule plus standard oncology risk-review practice. |
| Lymph nodes positive | Verify nodal stage and discuss systemic therapy/radiotherapy implications. | Nodal involvement is a major prognostic and treatment-planning factor in breast cancer. | Clinical oncology staging/treatment-planning principle; to be checked against local guidelines. |
| Grade 3 tumor | Treat as high-risk clinical feature in treatment planning. | Higher grade is associated with more aggressive biology and is commonly considered in adjuvant treatment decisions. | Clinical oncology risk-stratification principle. |
| Post-treatment follow-up | Regular history/physical and mammography follow-up. | ASCO follow-up guidance recommends history/physical examination and mammography after primary treatment. | ASCO follow-up guideline summary: https://pubmed.ncbi.nlm.nih.gov/23129741/ |
| BCSC high future-risk tier | Consider closer risk review and clinician-led prevention/screening discussion. | BCSC is a risk-stratification model, not a diagnosis model; high-risk groups may support screening/prevention discussion. | BCSC risk-model framing plus ASBrS endocrine risk-reduction statement: https://www.breastsurgeons.org/docs/statements/asbrs-endocrine-therapy.pdf |

## Important Clinical Note

All outputs are decision support only. They should not replace physician judgment, pathology review, IHC/FISH confirmation, multidisciplinary tumor board decisions, or current NCCN/ASCO/local clinical guidelines.
