# Novozymes Enzyme Stability Prediction
This is a project from a Kaggle competition hosted by Novozymes (more here: https://www.kaggle.com/competitions/novozymes-enzyme-stability-prediction), where the objective was to accurately predict the thermostability of an enzyme by only being given the nucleic sequence, the pH of the solution, and the reference to each of the protein in the dataset. The training dataset consisted of ~32k protein sequences and they were used to predict the melting temperature of a wildtype enzyme and its 2,413 variants.
The link to the IPYNB file can also be found here for easier viewing: https://www.kaggle.com/code/icyguy18/i190695. The work was not done alone, as various other contributions helped to identify the best procedure or ensemble of procedures.

# Working
Rather than taking the time to apply data cleaning to the training dataset, we employed an approach to directly take advantage of the testing dataset provided to us and use it to feature engineer new values that would aid us in predicting the melting temperature of the proteins. A series of configurations were tried on each of the remaining protein sequences, sometimes in ensemble:

1. Changes in Gibbs Free Energy were identified through a deep neural network DeepDDG, where the model extracted **ΔΔG** for each protein sequence with its wildtype. ΔΔG happens to have a high correlation with the melting temperature, as in general, ΔΔG tells a person how much energy is required to unfold or denature the very protein.
2. **Energy scores** were calculated by feeding protein sequences to the Rosetta framework and subsequently used to predict the melting temperature.
3. ThermoNet, another deep learning computation framework, was used to predict ΔΔG values for each of the protein sequence in the testing dataset. Moreover, these scores along with "FastRelax" protocol in Rosetta enabled an ensemble that yielded better predicted results.
4. AlphaFold V2 was used to generate PDB structures of the point mutation proteins of the wildtype protein, which already had a PDB file provided by Novozymes. These PDB files contain a Debye-Waller factor, or **b-factor** -- i.e. the uncertainty of the displacement of atoms from their positions -- that when used as parameter for confidence of prediction, happens to significantly correlate with the melting temperatures of the protein.
5. Various **substitution matrices** were used to calculate single-point substitution mutation scores. Apart from the commonly applied BLOSUM100 matrix, we tested with other substitution matrices, including but not limited to PAM1, LG, WAG, and WAG* matrices. In our findings, substitution matrices, while posing a positive correlation with the melting temperature, was not significant enough to have a noticeable effect in the overall results.

# Results
In the end, an ensemble of relaxed Rosetta scores, ΔΔG, B-factor, and LG substitution matrices were used. Each of the scores were ranked statistically and their scores were adjusted with a modified formula of sigmoid function:

# $$f(x) = 1 - {1 \over 1+e^{-x/s_f}}$$
