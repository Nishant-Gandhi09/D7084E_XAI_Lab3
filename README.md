# D7084E XAI Lab 3

This notebook implements the explanation and rule-building stages of Lab 3 for the CICIDS2017 intrusion-detection data:

1. **SHAP explanations** for a supervised detector and a semi-supervised detector.
2. **LIME explanations** for the same predictions used by SHAP.
3. **Explanation comparison** for faithfulness, stability, and readability.
4. **Feature selection for BRBES** using the top two SHAP features for each model.
5. **Belief rule base construction** for each detector using the selected features.

The notebook is now focused on the Lab 3 objectives and does not include the earlier extra Lab 2 experiments.

## Implemented so far

- Loads and combines the eight CICIDS2017 CSV files in 'data/'.
- Removes identifiers and timestamps that describe capture context rather than traffic behavior.
- Converts the original labels into a binary target: '0 = BENIGN', '1 = attack'.
- Removes duplicate flows and converts invalid numeric values to missing values.
- Creates fixed stratified train, validation, and test partitions using 'SEED = 42'.
- Trains a supervised Random Forest using all labelled training data.
- Trains a semi-supervised Random Forest with 10% labelled data and iterative pseudo-labelling.
- Uses median imputation inside each model pipeline and balanced Random Forest class weights.
- Uses the same three held-out test flows for both SHAP and LIME.
- Produces SHAP global bar plots, SHAP beeswarm plots, and local waterfall plots.
- Produces LIME local feature contributions and labels each contribution as pushing toward attack or benign.
- Compares SHAP and LIME for faithfulness, stability, and readability.
- Measures faithfulness by masking the top five features and recording the change in attack probability.
- Measures stability by repeating explanations and calculating top-feature overlap.
- Reports qualitative strengths and limitations of each explanation method.
- Ranks global feature importance using mean absolute SHAP values for both learning paradigms.
- Selects the two most important SHAP features for each model as inputs for the BRBES implementation.
- Checks the SHAP-selected features against the local LIME explanations as a local consistency check.
- Selects 'Destination Port' and 'Packet Length Std' for the supervised BRBES, and 'Destination Port' and 'Bwd Packet Length Std' for the semi-supervised BRBES.
- Builds a BRBES for each model using Low/Medium/High states for the selected features and data-derived Low/Medium/High risk beliefs.
- Stores explicit rule weights theta = 1 and attribute weights delta = 1 for every rule because no expert weighting was specified.
- Implements fuzzy input transformation, normalized rule activation, missing-input ignorance, ER-style belief aggregation, and a utility-based Attack/Benign decision.
- Evaluates the BRBES on the same explanation samples and reports risk beliefs, ignorance, utility, and the final decision.

## Requirements

Install the dependencies once in the notebook environment:

#python
%pip3 install -r requirements.txt
%pip3 install shap lime


The main libraries are 'pandas', 'numpy', 'scikit-learn', 'matplotlib', 'seaborn', 'shap', and 'lime'.

## Running the notebook

1. Open 'lab3.ipynb' in VS Code or Jupyter.
2. Keep the 'data/' directory beside the notebook.
3. Run the cells from top to bottom.
4. For a faster development run, set 'USE_SAMPLE = True' in the configuration cell. Use 'False' for the full experiment.

The full dataset contains approximately 2.8 million cleaned flows, so model training and pseudo-labelling can take substantial time. The notebook prints the pseudo-labelling history so the number of added and remaining rows can be checked.

## Interpretation notes

- In SHAP, positive values push the model toward the attack class and negative values push it toward benign.
- In LIME, positive local surrogate weights mean “toward attack”; negative weights mean “toward benign”.
- The global SHAP bar plot ranks features by average absolute impact. The beeswarm plot also shows direction and feature values.
- SHAP and LIME are local/global explanation tools, not replacements for evaluating detector accuracy.

## Current observed explanation pattern

The completed run produced correct predictions for the three shared explanation samples: two benign flows and one attack flow for each paradigm. Both models repeatedly identified 'Destination Port', packet-size features, and packet-length statistics among the strongest explanations, although local LIME feature ranges vary by sample.

## Reproducibility

- Random seed: '42'
- Explanation samples: '3' fixed test rows selected with 'random_state=42'
- Pseudo-labelling confidence threshold: '0.95'
- Maximum pseudo-labelling rounds: '10'
- Semi-supervised labelled budget: '10%'

## Step 4 comparison results

The comparison uses the same three local test flows for both models. In the completed run, SHAP selected the same top features on repeated runs for every sample, giving a stability overlap of 1.0. LIME had lower but non-zero overlap, ranging from approximately 0.43 to 0.67, because it fits a stochastic local surrogate.

The masking experiment showed larger attack-probability changes for SHAP on these samples, indicating stronger measured local faithfulness for this run. LIME remained useful because its interval-based rules are compact and easy to read. The methods generally agreed on prediction direction and recurring features such as 'Destination Port', protocol, and packet-length features, but disagreements are expected when correlated features share influence or when the local surrogate approximates a nonlinear Random Forest decision.

## Step 5 conclusion

The feature selection for BRBES are based on the results from the SHAP analysis.
Our global SHAP anaylsis identified which two features was of most importance for each model, for supervised we found that Destination Port and Packet Length Std was of most importance and for semi-supervised we found that Destination Port and Bwd Packet Length Std was the important features.
These features was then checked against the local LIME explanations. As seen in the table above the destination port appeared in all three local Lime explanations for the supervised learning, while the packet length did not appear among the local Lime explanations.
For the semi-supervised model both our features from the SHAP analysis appeared in the local LIME explanation.
This partial disagreement we could see for the supervised model is not unexpected, since SHAP ranks the features using global importance across the balanced evalutation sample, while LIME explains it locally.
The feature selection for BRBES is therefore based on the gliabl SHAP ranking and we're using LIME as a local consistence check.

## Step 6 BRB construction

The final stage creates one BRBES per model using the selected SHAP features. Each feature is transformed into Low/Medium/High matching degrees using training-data cut points. For every feature-state combination, the rule stores a three-level risk belief distribution derived from the training attack share, rule weight theta = 1, and attribute weights delta = 1. The inference stage normalizes rule activation, preserves missing-input ignorance, combines the evidence with ER-style aggregation, and converts the final risk distribution into a utility score. The 33rd and 67th percentiles are only a transparent three-bin design choice; they are not stated as mandatory values in the lab sheet.
