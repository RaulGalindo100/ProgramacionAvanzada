# MotifGate: A PWM-conditioned TFBS prediction model with biologically realistic negatives in short sequences.

## Overview

MotifGate is a predictive framework for transcription factor binding site (TFBS) classification in short DNA sequence windows. The model is designed for the site-level discrimination of short sequences associated with transcription factor motifs, with a particular focus on windows of length K = 7 to K = 14. Unlike purely neural sequence classifiers, MotifGate explicitly incorporates a position weight matrix (PWM) prior and learns a residual correction over this motif-informed baseline.

The central hypothesis is that a PWM provides a strong and interpretable prior for TFBS recognition, but that additional sequence-dependent information can improve discrimination under hard-negative evaluation. MotifGate, therefore, combines an exactly differentiable PWM prior with a learned residual branch modulated by a trainable gate. This design allows the model to quantify when learned sequence information improves over a motif prior and when the PWM alone remains sufficient.

The project evaluates MotifGate on JASPAR dataset benchmarks using leakage-aware splitting, hard-negative sampling, family and class-based group evaluation, and interpretability analyses.

---

## Main objectives

The project aims to:

1. Develop a PWM-aware neural architecture for short-window TFBS prediction.
2. Evaluate the model under biologically realistic hard-negative settings.
3. Avoid sequence leakage through canonical reverse-complement deduplication.
4. Compare performance across different short sequence lengths K.
5. Analyze generalization across TF families and TF classes.
6. Quantify the contribution of the learned residual branch over the PWM prior.
7. Include interpretability and validate it using attribution and perturbation analyses.

---

## Problem formulation

Given a short DNA sequence \(s\) of fixed length \(K\) and a transcription factor motif \(m\), the task is to predict whether \(s\) corresponds to a binding site for motif \(m\).

Formally, the model estimates:

\[
P(y=1 \mid s, m),
\]

where:

- \(s \in \{A,C,G,T\}^K\) is a short DNA sequence.
- \(m\) is a motif identifier associated with a PWM.
- \(y=1\) indicates a positive TFBS instance.
- \(y=0\) indicates a negative instance, typically sampled from binding sites of other TFs.

The project focuses on **site-level binary classification**, not genome-wide peak calling or chromatin-aware TF binding prediction.

---

## Dataset

The benchmark is constructed from JASPAR motif and site information.

### Positive examples

Positive examples correspond to TFBS-associated sequences derived from JASPAR motif/site data. Each example is associated with:

- DNA sequence
- Motif ID
- TF family
- TF class
- Positive label

Sequences are processed into fixed-length windows for each value of \(K\).

### Negative examples

Negative examples are generated using hard-negative sampling. Instead of using random genomic background, negatives are sampled from binding sites of other transcription factors. This makes the task more biologically realistic because the model must distinguish a target TFBS from other TFBS-like sequences. The default setting uses a positive-to-negative ratio of approximately 1:2. 


### Sequence lengths

The project evaluates short-window TFBS prediction for multiple sequence lengths:
K = 7, 8, 9, 10, 11, 12, 13, 14


##  Evaluation protocols
MotifGate is evaluated under group protocols designed to test generalization beyond memorized motifs.
### Family-based evaluation
In family-based evaluation, the split structure considers TF families. This protocol evaluates whether the model generalizes across groups of transcription factors sharing related DNA-binding properties.
### Class-based evaluation
In class-based evaluation, the split structure considers broader TF structural classes. This setting is typically harder because TF classes can contain diverse families and heterogeneous motif structures.

## Model architecture
MotifGate combines two main components:
A PWM prior branch.
A learned residual branch.
The final prediction is based on a gated combination of motif-informed scoring and neural residual correction.
### PWM prior
For each motif mmm, a PWM score is computed for the input sequence sss. This score acts as an explicit motif-aware prior.
The PWM branch provides a strong baseline because it directly encodes known nucleotide preferences at motif positions.
### Residual branch
The residual branch learns sequence-dependent corrections that are not fully captured by the PWM prior. This branch can exploit additional information, including:
short-range nucleotide dependencies;
flanking sequence context;
motif extensions;
class- or family-specific residual patterns;
deviations from the canonical PWM signal.
### Residual gate
A key contribution of MotifGate is the residual gate. The gate controls the contribution of the learned residual correction relative to the PWM prior.

Conceptually, the prediction logit can be written as:
z(s,m)=zPWM(s,m)+g⋅rθ(s,m),z(s,m) = z_{\text{PWM}}(s,m) + g \cdot r_{\theta}(s,m),z(s,m)=zPWM​(s,m)+g⋅rθ​(s,m),
where:
zPWM(s,m)z_{\text{PWM}}(s,m)zPWM​(s,m) is the PWM prior logit or standardized PWM score.
rθ(s,m)r_{\theta}(s,m)rθ​(s,m) is the learned residual correction.
ggg is a trainable residual gate.
z(s,m)z(s,m)z(s,m) is the final prediction logit.
The gate makes the architecture interpretable because it allows the model to quantify how much learned residual information is being used beyond the motif prior.

## Architectural components
Depending on the experimental configuration, MotifGate includes:
PWM-aware input features.
Convolutional sequence encoders.
Residual correction layers.
Cross-attention modules between sequence and motif-derived representations.
Mixture-of-experts components.
Squeeze-and-excitation mechanisms.
Auxiliary losses encourage consistency with the PWM prior.

## Training
The model is trained as a binary classifier using positive TFBS examples and hard negatives.
epochs: 50
patience: 5
batch size: 256
learning rate: 2e-3
weight decay: 1e-4
dropout: 0.20
label smoothing: 0.05
negative ratio: 2 negatives per positive


## Metrics
The main evaluation metrics are:
### Average Precision
Average Precision is the primary metric because the task is binary classification with an imbalanced positive/negative structure.
AP=∑n(Rn−Rn−1)PnAP = \sum_n (R_n - R_{n-1}) P_nAP=n∑​(Rn​−Rn−1​)Pn​
where PnP_nPn​ and RnR_nRn​ are precision and recall at threshold nnn.
### AUROC
The area under the receiver operating characteristic curve measures ranking quality across positive and negative examples.
AUROC=P(f(x+)>f(x−)).AUROC = P(f(x^+) > f(x^-)).AUROC=P(f(x+)>f(x−)).

## Interpretability
MotifGate includes interpretability analyses designed to evaluate whether the model uses biologically meaningful sequence information.
### Integrated Gradients
Integrated Gradients is used to assign attribution scores to nucleotide positions. These attributions indicate which positions contribute most strongly to the prediction.
### PWM-information comparison
Attribution patterns can be compared with motif information content. This helps evaluate whether the model focuses on positions consistent with motif-informative regions.
### Perturbation validation
Important positions identified by attribution methods are perturbed or mutated. If the model is using meaningful sequence signal, perturbing high-attribution positions should reduce the predicted binding probability more strongly than perturbing low-attribution positions.
### Sanity checks
Sanity checks are used to test whether attribution maps reflect learned model structure rather than artifacts. These may include:
Parameter randomization tests.
Label randomization tests.
PWM-shuffled controls.
Attribution and performance correlation analyses.
Together, these analyses help validate that MotifGate’s residual branch learns interpretable sequence features beyond the PWM prior.






