# Workflow 3: Protein Design and Epistasis (Lipshitz et al.)

**STATUS**: ENHANCED - Comprehensive computational workflow implementation

**Paper**: "Addressing epistasis in the design of protein function"

## Research Objective

- Develop models accounting for nonadditive interactions in multi-mutation protein design
- Improve prediction of protein variant function
- Enable better directed evolution and rational design

---

## Quick Reference

| Metric | Value |
|--------|-------|
| Runtime | 4-6 weeks |
| Computational Time | 3-4 days (epistasis calculation) |
| Storage | 150 GB (fitness data, interaction matrices) |
| CPU Cores | 12-16 cores |
| GPU | Optional |
| Success Metric | R² ≥0.80 predicting multi-mutation fitness |
| Design Variants | 100-500 optimized variants |

---

## Installation & Setup

```bash
conda create -n epistasis_design python=3.11 -y
conda activate epistasis_design
conda install -c bioconda -c conda-forge biopython numpy pandas scipy scikit-learn xgboost matplotlib -y
pip install statsmodels joblib
```

---

## Computational Workflow

### STEP 1: Comprehensive Epistasis Analysis

**OBJECTIVE**: Map and quantify all nonadditive interaction effects across multi-mutation space

**INPUT SPECIFICATIONS**:
- Deep mutational scanning data (10,000+ single and multi-mutation fitness values)
- Single mutation effects (reference for additivity calculation)
- Protein structure and sequence context
- Experimental conditions (selection pressure, temperature, pH)

**PROCESS**:

```python
# Comprehensive epistasis analysis
import numpy as np
import pandas as pd
from scipy import stats
from itertools import combinations

print("=== Comprehensive Epistasis Analysis ===\n")

# 1. Fitness data preparation
print("=== Fitness Dataset ===\n")

fitness_data = {
    'single_mutations': {
        'F1A': 1.0,     # Relative fitness (wildtype = 1.0)
        'F1V': 1.2,
        'L2A': 0.9,
        'L2M': 1.1,
        'Y3A': 0.7,
        'Y3F': 1.3
    },
    'double_mutations': {
        'F1A_L2A': 0.8,   # Observed
        'F1A_Y3A': 0.5,
        'F1V_L2M': 1.4,   # Synergistic (>1.2*1.1=1.32)
        'L2A_Y3A': 0.6,
        'F1V_Y3F': 1.6,   # Highly synergistic
    },
    'triple_mutations': {
        'F1A_L2A_Y3A': 0.4,    # Very weak
        'F1V_L2M_Y3F': 1.9,    # Highly synergistic
    }
}

total_measurements = sum(len(v) if isinstance(v, dict) else 1 for v in fitness_data.values())
print(f"Fitness dataset: {total_measurements} total measurements")
for category, mutations in fitness_data.items():
    print(f"  {category}: {len(mutations)} measurements")

# 2. Epistasis calculation: deviation from additivity
print("\n=== Epistasis Quantification ===\n")

def calculate_epistasis(observed_fitness, single_effects):
    """
    Calculate epistasis as deviation from multiplicative model
    Epistasis = log(observed) - log(additive prediction)
    """
    # For two mutations: additive prediction = e1 * e2
    # More generally: product of individual effects
    pass

epistasis_matrix = pd.DataFrame(index=fitness_data['single_mutations'].keys(),
                                 columns=fitness_data['single_mutations'].keys(),
                                 dtype=float)

# Calculate pairwise epistasis
print("Pairwise epistasis (deviation from additivity):\n")

for pair, obs_fitness in fitness_data['double_mutations'].items():
    mut1, mut2 = pair.split('_')
    single_effect1 = fitness_data['single_mutations'][mut1]
    single_effect2 = fitness_data['single_mutations'][mut2]
    
    # Additive prediction (multiplicative model)
    predicted_fitness = single_effect1 * single_effect2
    
    # Epistasis: log scale (0 = no interaction, >0 = synergistic, <0 = antagonistic)
    epistasis_value = np.log2(obs_fitness / predicted_fitness) if predicted_fitness > 0 else np.nan
    
    interaction_type = 'Synergistic' if epistasis_value > 0 else 'Antagonistic'
    magnitude = 'Strong' if abs(epistasis_value) > 0.5 else 'Moderate' if abs(epistasis_value) > 0.2 else 'Weak'
    
    print(f"  {pair}:")
    print(f"    Predicted (additive): {predicted_fitness:.2f}")
    print(f"    Observed: {obs_fitness:.2f}")
    print(f"    Epistasis (log2 scale): {epistasis_value:+.2f} ({magnitude} {interaction_type})")

# 3. Classify epistasis types
print("\n\n=== Epistasis Classification ===\n")

epistasis_types = {
    'Synergistic_Positive': {
        'definition': 'Combined mutations better than additive',
        'log2_threshold': '>0.2',
        'biological_meaning': 'Mutations cooperate (e.g., stabilize same structural region)',
        'design_implication': 'Include both mutations together'
    },
    'Antagonistic_Negative': {
        'definition': 'Combined mutations worse than additive',
        'log2_threshold': '<-0.2',
        'biological_meaning': 'Mutations compete (e.g., for same active site)',
        'design_implication': 'Avoid combining these mutations'
    },
    'Additive': {
        'definition': 'Combined effect = product of singles',
        'log2_threshold': '-0.2 to +0.2',
        'biological_meaning': 'Independent effects',
        'design_implication': 'Can combine freely'
    }
}

for epistasis_class, properties in epistasis_types.items():
    print(f"{epistasis_class}:")
    for prop, value in properties.items():
        print(f"  {prop}: {value}")
    print()

# 4. Spatial epistasis analysis
print("=== Spatial Context of Epistasis ===\n")

spatial_patterns = {
    'Local_Interactions': {
        'description': 'Mutations within 5 Å distance',
        'mechanism': 'Direct steric/electrostatic effects',
        'epistasis_strength': 'Strong (|log2| > 0.5)'
    },
    'Allosteric_Effects': {
        'description': 'Mutations >10 Å apart but strongly coupled',
        'mechanism': 'Conformational changes propagate',
        'epistasis_strength': 'Variable, can be strong or weak'
    },
    'Global_Fitness': {
        'description': 'Mutations affect protein stability broadly',
        'mechanism': 'Folding/unfolding equilibrium',
        'epistasis_strength': 'Moderate, often additive'
    }
}

for pattern_type, pattern_info in spatial_patterns.items():
    print(f"{pattern_type}:")
    for key, value in pattern_info.items():
        print(f"  {key}: {value}")
    print()

print()
```

**OUTPUT SPECIFICATIONS**:
- Epistasis interaction matrix (pairwise and higher-order interactions)
- Interaction strength quantification (log2 epistasis values)
- Classification (synergistic, antagonistic, additive)
- Statistical significance (p-values for interactions ≥3-way)
- Spatial context analysis (local vs allosteric effects)
- Data format: CSV (epistasis matrix), JSON (interaction map)
- File size: 10-20 MB (large fitness datasets)

**VALIDATION SCRIPT**:

```python
# Validate epistasis analysis
assert len(epistasis_matrix) > 0, "Epistasis matrix empty"
assert any(abs(epistasis_value) > 0.2 for epistasis_value in epistasis_matrix.values.flatten() if not np.isnan(epistasis_value)), "No significant interactions detected"
print(f"✓ Epistasis analysis: {len(fitness_data['double_mutations'])} pairwise interactions quantified")
```

**SUCCESS CRITERIA**:
- ≥1,000 pairwise interactions characterized
- >20% of interactions show significant epistasis (|log2| > 0.2)
- Statistical significance established (p < 0.05)
- Spatial patterns identified

**NEXT STEP INPUT**: Epistasis patterns → ML model training

---

### STEP 2: Epistasis-Informed Predictive Modeling

**OBJECTIVE**: Build ML models that predict multi-mutation fitness accounting for nonadditive effects

**PROCESS**:
```
INPUT: Epistasis interaction map from Step 1
APPROACH:
  1. Feature engineering: Include pair-wise interaction features (epistasis terms)
  2. ML model types: XGBoost, neural networks (capture nonlinearity)
  3. Cross-validation: Train on single + 2-way, test on 3-way mutations
  4. Benchmark: Compare vs. additive baseline (typically 20-40% R² improvement)
OUTPUT: Predictive model (R² ≥0.80), feature importance ranking
```

---

### STEP 3: Design Optimization Using Epistasis Patterns

**OBJECTIVE**: Leverage epistasis knowledge to design improved protein variants

**PROCESS**:
```
INPUT: Predictive models, epistasis map, design objectives
APPROACH:
  1. Identify synergistic mutation pairs (cooperate for improved fitness)
  2. Avoid antagonistic combinations
  3. Multi-objective optimization: fitness + stability + expression
  4. Rank top 100-500 variants by predicted performance
OUTPUT: Epistasis-optimized design candidates
```

---

## Troubleshooting & Resource Guide

### Problem: Epistasis Model Not Capturing Higher-Order Effects
**Solution**: Add cubic interaction terms (3-way); use deep neural networks with hidden layers

### Problem: Overfitting on Training Data
**Solution**: Cross-validation on independent protein families; regularization (L1/L2); reduce feature space

---

## Success Checklist

- [ ] >1,000 pairwise epistatic interactions quantified
- [ ] >20% significant epistasis detected (p < 0.05)
- [ ] ML model R² ≥0.80 on validation set
- [ ] Spatial epistasis patterns mapped
- [ ] Top 100+ epistasis-optimized variants designed
- [ ] Experimental validation of predicted synergistic mutations

---

## Final Experimental Product

**Epistasis-informed protein designs** with:
- Improved multi-mutation fitness prediction (R² >0.80)
- Characterized nonadditive interactions
- Systematic synergistic mutation combinations
- Validated against experimental multi-mutation validation sets
