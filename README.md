# Continual Learning: Named Entity Recognition using BERT
## Task Overview
The task is a hands-on continual learning evaluation using the [**Facebook Clinical Trial**](https://github.com/facebookresearch/Clinical-Trial-Parser) dataset for Named Entity Recognition (NER). The goal is to train models sequentially across multiple datasets while avoiding catastrophic forgetting and improving knowledge transfer. Complete details can be found in "**Continual learning task.pdf**".

**📂 Dataset**

We focus on NER with entities such as **Treatment, Chronic Disease, Cancer, Allergy, Other**, etc. Three datasets (G1, G2, G3) are provided, defining three tasks (T1, T2, T3) with the same label set but different entities.

**🏗️ Task Breakdown**

- Train sequentially:

  - Step 1: Train on T1 (G1).

  - Step 2: Train on T2 (G2), keeping only 100 examples from T1.

  - Step 3: Train on T3 (G3), keeping only 100 examples each from T1 & T2.

- Evaluate performance on T1, T2, T3 test sets.

- Compare with a baseline model trained on combined dataset (G1+G2+G3).

- Metrics: Entity-wise and Weighted F1 scores.

- Provide code to evaluate on a new unseen task (T4 with dataset G4) using:
  - 100 retained examples from past tasks
  - Full dataset G4

- Report all metrics after training on T4.

## Approach

**Data Preprocessing**

- Split into train (80%) / test (10%) / validation (10%).

- Standardized tags, tokenized using BERT/Roberta tokenizers, aligned word-level and token-level labels.

- For continual learning (T2, T3), included 100 examples from previous tasks using stratified sampling to preserve label distribution.

**Model Training**

- Tested multiple models:

  - Baseline: AutoModelForTokenClassification (BERT-base-uncased + dense layer).
    - Poor performance, combined model outperformed T3.

  - Custom Models: BERT-base with custom 3-layer DNN (512→128→5).
     -Improved performance.

  - Biomedical Pretrained Models: Biomed-RoBERTa, PubMedBERT, BioBERT, Biomedical-NER.
    - Tried but only partially trained due to GPU limits.

  - Best Models: All used **Bio_ClinicalBERT** as the base transformer. We tried 2 variants for the DNN post transformer and the 2nd variant gave the best results
    - 1) Sum of last 4 hidden states → DNN with batchnorm + dropout.
    - 2) Concatenation of last 2 hidden states → same DNN.

Notably, best T3 model using the 2nd variant described above outperformed combined model trained with G1+G2+G3 datasets with "Best Models" strategy described above.

**Training Setup**:

- Optimizer: AdamW

- Learning rate: 3e-4

- Batch size: 8 (limited by GPU)

- Epochs: 20

- Loss: Cross-Entropy

- Early stopping with F1-based validation metrics (weighted F1 or sum of key F1 scores).

**Task T4 Generalization**

Code provided for re-training best model on unseen dataset (T4) using 100 retained samples + G4 dataset.

## Results

- Baseline BERT + AutoModelForTokenClassification → Poor results.

- Custom DNN + BERT → Better, but still suboptimal.

- Bio_ClinicalBERT with enhanced DNN (sum/concat of hidden states, batchnorm, dropout) →
  - Best performing approach.
  - T3 continual learning model outperformed combined dataset model → shows effective knowledge retention and transfer.

- Training code and detailed results are in NER.ipynb and Results.xlsx.

## Suggested Improvements

- Increase batch size with larger GPUs.

- Hyperparameter tuning (LR, optimizer, patience).

- Try larger biomedical models (e.g., BioGPT).

- Experiment with freezing transformer layers to reduce forgetting.

- Explore different subset sampling strategies (bias towards rare tags or low-performing labels).

= Modify validation metrics.

- Weighted loss functions (e.g., Loss_T2 = 1.2·Loss_T1 + Loss_T2).

- Use ensemble models for better generalization.

- Add heuristics for common entities (diseases, treatments, allergies).

