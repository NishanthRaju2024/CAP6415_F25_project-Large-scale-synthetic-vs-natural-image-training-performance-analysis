# CAP6415_F25_project-Large-scale-synthetic-vs-natural-image-training-performance-analysis

1. ABSTRACT
--------------------------------------------------------------------------------
This study analyzes the performance impact of training State-of-the-Art (SOTA) 
computer vision models using Natural images versus Synthetic (Simulated) images. 
We evaluated two distinct modern architectures—ConvNeXt-Tiny (CNN) and ViT-B/16 
(Vision Transformer)—across three data strategies: Real Only, Synthetic Only, 
and a Mixed/Curriculum approach.

Key Findings:
* Naive Mixing Fails: Simply combining real and synthetic data (50/50 split) 
  proved highly unstable. While it peaked at 94.4%, it crashed to 88.9% in later 
  epochs, confirming the existence of "Negative Transfer" due to conflicting 
  feature distributions.
* Curriculum Learning Succeeds: A two-stage "Sim-to-Real" pipeline stabilized 
  training and achieved the highest accuracy (96.7% for ConvNeXt, 97.5% for 
  ViT), proving that synthetic pre-training optimizes even modern SOTA models.


2. EXPERIMENTAL METHODOLOGY
--------------------------------------------------------------------------------
2.1 Models Selected
We utilized two SOTA architectures to represent different inductive biases:
1. ConvNeXt-Tiny: A modern Convolutional Network designed to compete with 
   Transformers. It represents the SOTA for pure CNNs but retains an inductive 
   bias toward local textures.
2. ViT-B/16: A Vision Transformer that relies on global self-attention. It is 
   typically shape-biased and data-hungry.

2.2 Data Strategy
To ensure reproducibility and isolate the "Sim-to-Real" domain gap, we utilized 
the CIFAR-10 dataset with the following transformations:
* Natural Data: Standard CIFAR-10 images with normalization.
* Synthetic Data: Real images mathematically processed to mimic generative 
  artifacts:
    - Gaussian Blur (sigma=0.1-2.0): Simulates the "smooth/waxy" texture 
      common in GAN/Diffusion outputs.
    - Color Jitter: Simulates lighting and tone inconsistencies.


3. PERFORMANCE ANALYSIS
--------------------------------------------------------------------------------
3.1 Baseline Comparison (Real vs. Synthetic)
We first established baselines by training models exclusively on one data type.

- ConvNeXt (Real Only): 95.8% Accuracy. High performance but showed volatility.
- ConvNeXt (Synthetic Only): 95.5% Accuracy. 
  > Analysis: The synthetic model was surprisingly competitive, nearly matching 
    the real baseline. The noise/blur acted as a shape regularizer.

3.2 The "Mixed Data" Failure
We tested a naive combination strategy (50% Real + 50% Synthetic).
- Result: Unstable. Peak 94.4% -> Crashed to 88.9%.
- Analysis: This highlights "Negative Transfer." The model received conflicting 
  gradient signals—one set pushing for texture sensitivity (Real) and another 
  for texture invariance (Synthetic). This caused the model to diverge in later 
  epochs.

3.3 The "Curriculum Learning" Solution
We implemented a Sim-to-Real transfer pipeline: 
Stage 1 (Synthetic Pre-training) -> Stage 2 (Real Fine-tuning).

- ConvNeXt-Tiny Result: 96.7% (+0.9% over Real Baseline; +7.8% over Mixed Crash).
- ViT-B/16 Result: 97.5% (SOTA Performance).

Why it worked:
1. Stage 1 (Shape Learning): The synthetic data forced the models to learn 
   robust topological features (e.g., "wheels," "wings") without relying on 
   easy texture shortcuts.
2. Stage 2 (Texture Adaptation): The fine-tuning phase aligned these robust 
   features with the specific pixel statistics of the real-world camera domain.


4. VISUAL EVIDENCE
--------------------------------------------------------------------------------
Please refer to 'results/sota_final_analysis.png' in the repository.
The training graph illustrates the divergence between strategies. Note the 
Cyan line (Mixed) becoming unstable and crashing, while the Bold Blue line 
(Curriculum) jumps in accuracy at the Stage Switch (Epoch 5) and remains stable.


5. CONCLUSION
--------------------------------------------------------------------------------
This analysis demonstrates that Synthetic Data is not just "more data"—it is 
a distinct domain.

1. Texture vs. Shape: Synthetic training promotes shape robustness but lacks 
   texture fidelity.
2. Optimization Strategy: Naive mixing causes instability in SOTA CNNs. A 
   Curriculum (Sim-to-Real) approach is required to leverage synthetic data 
   effectively.
3. Architecture Robustness: Both ConvNeXt and ViT benefited from the pipeline, 
   achieving peak performance (96.7% and 97.5%) only when using the Curriculum 
   strategy.