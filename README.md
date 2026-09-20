# D7084E XAI Lab 3

This notebook implements the explanation and rule-building stages of Lab 3 for the CICIDS2017 intrusion-detection data:

1. **SHAP explanations** for a supervised detector and a semi-supervised detector.
2. **LIME explanations** for the same predictions used by SHAP.
3. **Explanation comparison** for faithfulness, stability, and readability.
4. **Feature selection for BRBES** using the top two SHAP features for each model.
5. **Belief rule base construction** for each detector using the selected features.

The notebook is now focused on the Lab 3 objectives and does not include the earlier extra Lab 2 experiments.

## Requirements

Install the dependencies once in the notebook environment:

#python
%pip3 install -r requirements.txt
%pip3 install shap lime


The main libraries are 'pandas', 'numpy', 'scikit-learn', 'matplotlib', 'seaborn', 'shap', and 'lime'.

## Running the notebook

1. Open 'lab3.ipynb' in VS Code or Jupyter.
2. Create 'data/' directory in root and enter the CICIDS2017 dataset. 
3. Run the cells from top to bottom.
4. For a faster development run, set 'USE_SAMPLE = True' in the configuration cell. Use 'False' for the full experiment.

The full dataset contains approximately 2.8 million cleaned flows, so model training and pseudo-labelling can take substantial time. The notebook prints the pseudo-labelling history so the number of added and remaining rows can be checked.


## Reproducibility
- Random seed: '42'
- Explanation samples: '3' fixed test rows selected with 'random_state=42'
- Pseudo-labelling confidence threshold: '0.95'
- Maximum pseudo-labelling rounds: '10'
- Semi-supervised labelled budget: '10%'
- Final data mode: 'USE_SAMPLE = False' (full cleaned dataset)