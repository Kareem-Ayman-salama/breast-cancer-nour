# Breast Cancer AI Decision Support

Clean two-path breast cancer decision-support project.

## What Is Included

- Streamlit dashboard for:
  - BCSC pre-diagnosis risk assessment
  - METABRIC post-diagnosis subtype/survival/recurrence support
  - doctor dashboard
  - evidence, figures, recommendation knowledge base, and model comparisons
- Clean notebook:
  - `deliverables/CLEAN_Two_Path_Breast_Cancer_AI_Complete.ipynb`
- Final Arabic client report:
  - `deliverables/CLEAN_Final_Client_Report_AR.docx`
- Clean output summaries, figures, and benchmark CSV/JSON files.

Raw external datasets are not included in this repository.

## Run Streamlit

```powershell
pip install -r streamlit_decision_support_app/requirements.txt
streamlit run streamlit_decision_support_app/app.py
```

The app expects the included output files under:

```text
outputs/clean_two_path_complete
```

## Key Results

- BCSC ROC-AUC: `0.649`
- BCSC top 10% risk lift: `2.17x`
- METABRIC subtype balanced accuracy: `0.749`
- METABRIC 5-year survival AUC: `0.789`
- METABRIC 5-year recurrence AUC: `0.767`
- Subtype top-2 accuracy: `94.2%`
- ER/PR/HER2 decision-support tasks: `98-99%+` accuracy/AUC range

## Clinical Note

This project is for research and decision support only. It does not replace physician judgment, pathology review, IHC/FISH confirmation, or current clinical guidelines.
