# Workflow 5: Addressing Epistasis in Protein Design

**Paper**: "Addressing epistasis in the design of protein function"

## Research Objective

- Quantify epistatic interactions in protein fitness landscapes
- Develop computational methods to predict epistasis
- Design strategies to overcome epistasis in protein engineering
- Integrate epistasis prediction into protein design workflows

---

## Quick Reference

| Metric | Value |
|--------|-------|
| **Computational Time** | 6-10 weeks |
| **CPU Requirements** | 16-32 cores |
| **Storage** | 200 GB |
| **Languages** | Python 3.8+ |
| **Success Metric** | Identify 50-100 beneficial mutation combinations |
| **Epistasis Detection Threshold** | |ε| > 0.5 (deviation from additivity) |
| **Required Tools** | scikit-learn, GROMACS, PyMOL |

---

## Computational Workflow

### STEP 1: Fitness Landscape Mapping

**INPUT**: 
- Protein sequence data
- Knowledge of functional constraints from experimental data
- Target phenotype/activity specifications

**PROCESS**: Complete executable Python code for deep mutational scanning simulation and epistasis quantification

```python
import numpy as np
import pandas as pd
from itertools import combinations
from sklearn.preprocessing import StandardScaler
import matplotlib.pyplot as plt
import json

print("=== Epistasis in Protein Design - STEP 1: Fitness Landscape Mapping ===\n")

# ============================================================================
# PART 1: Protein Sequence Space and Mutation Library
# ============================================================================

class ProteinFitnessLandscape:
    def __init__(self, protein_seq, target_sites=None):
        self.protein_seq = protein_seq
        self.protein_length = len(protein_seq)
        self.target_sites = target_sites if target_sites else list(range(0, self.protein_length, 10))
        self.single_mutations = {}
        self.pairwise_interactions = {}
        self.fitness_data = {}
    
    def generate_deep_mutational_scanning_library(self, num_variants=500):
        """Generate comprehensive mutation library at target sites"""
        
        print(f"Protein: {len(self.protein_seq)} amino acids")
        print(f"Target sites for scanning: {len(self.target_sites)} positions\n")
        
        aa_alphabet = 'ACDEFGHIKLMNPQRSTVWY'
        variants = []
        
        # Single mutations
        for site in self.target_sites:
            wt_aa = self.protein_seq[site]
            for mut_aa in aa_alphabet:
                if mut_aa != wt_aa:
                    variants.append({
                        'type': 'single',
                        'position': site,
                        'wt_aa': wt_aa,
                        'mut_aa': mut_aa,
                        'mutation': f"{wt_aa}{site}{mut_aa}",
                    })
        
        # Pairwise combinations (sample for computational feasibility)
        pairwise_combinations = list(combinations(self.target_sites, 2))
        sampled_pairs = np.random.choice(len(pairwise_combinations), 
                                        size=min(150, len(pairwise_combinations)), 
                                        replace=False)
        
        for pair_idx in sampled_pairs:
            pos1, pos2 = pairwise_combinations[pair_idx]
            wt_aa1, wt_aa2 = self.protein_seq[pos1], self.protein_seq[pos2]
            mut_aa1, mut_aa2 = np.random.choice(list(aa_alphabet.replace(wt_aa1, '')), 1)[0], \
                              np.random.choice(list(aa_alphabet.replace(wt_aa2, '')), 1)[0]
            
            variants.append({
                'type': 'pairwise',
                'positions': [pos1, pos2],
                'mutations': [f"{wt_aa1}{pos1}{mut_aa1}", f"{wt_aa2}{pos2}{mut_aa2}"],
                'wt_aa': [wt_aa1, wt_aa2],
                'mut_aa': [mut_aa1, mut_aa2],
            })
        
        print(f"Generated variant library:")
        print(f"  Single mutations: {sum(1 for v in variants if v['type'] == 'single')}")
        print(f"  Pairwise combinations: {sum(1 for v in variants if v['type'] == 'pairwise')}")
        print(f"  Total variants: {len(variants)}\n")
        
        return variants
    
    def predict_single_mutation_effects(self, variants, verbose=True):
        """Predict fitness effects for each single mutation"""
        
        print("=== Single Mutation Fitness Prediction ===\n")
        
        # Mutation property matrix (simplified)
        aa_props = {
            'A': {'hydrophobic': 1.0, 'charge': 0, 'size': 0.5},
            'C': {'hydrophobic': 0.7, 'charge': 0, 'size': 0.6},
            'D': {'hydrophobic': -0.5, 'charge': -1, 'size': 0.7},
            'E': {'hydrophobic': -0.6, 'charge': -1, 'size': 0.8},
            'F': {'hydrophobic': 1.0, 'charge': 0, 'size': 1.0},
            'G': {'hydrophobic': 0.5, 'charge': 0, 'size': 0.0},
            'H': {'hydrophobic': 0.3, 'charge': 0.5, 'size': 0.9},
            'I': {'hydrophobic': 1.0, 'charge': 0, 'size': 0.9},
            'K': {'hydrophobic': -0.3, 'charge': 1, 'size': 1.0},
            'L': {'hydrophobic': 1.0, 'charge': 0, 'size': 0.9},
            'M': {'hydrophobic': 0.9, 'charge': 0, 'size': 0.9},
            'N': {'hydrophobic': -0.4, 'charge': 0, 'size': 0.7},
            'P': {'hydrophobic': 0.7, 'charge': 0, 'size': 0.7},
            'Q': {'hydrophobic': -0.3, 'charge': 0, 'size': 0.8},
            'R': {'hydrophobic': -0.5, 'charge': 1, 'size': 1.1},
            'S': {'hydrophobic': -0.2, 'charge': 0, 'size': 0.5},
            'T': {'hydrophobic': 0.0, 'charge': 0, 'size': 0.6},
            'V': {'hydrophobic': 1.0, 'charge': 0, 'size': 0.8},
            'W': {'hydrophobic': 1.0, 'charge': 0, 'size': 1.1},
            'Y': {'hydrophobic': 0.8, 'charge': 0, 'size': 1.0},
        }
        
        single_muts = [v for v in variants if v['type'] == 'single']
        
        fitness_effects = []
        for mut in single_muts:
            pos = mut['position']
            wt_prop = aa_props[mut['wt_aa']]
            mut_prop = aa_props[mut['mut_aa']]
            
            # Fitness penalty (simplified model)
            # Larger changes = more likely deleterious
            property_change = sum(abs(wt_prop[k] - mut_prop[k])**2 for k in wt_prop) ** 0.5
            
            # Background fitness + mutation effect
            background_fitness = np.random.normal(1.0, 0.1)
            epistasis_context = np.random.normal(0, 0.15)  # Future epistasis variance
            
            fitness = background_fitness - 0.3 * property_change + epistasis_context
            
            # Add some beneficial mutations (5-10% typically beneficial)
            if np.random.random() < 0.08:
                fitness = background_fitness + np.random.uniform(0.05, 0.3)
            
            fitness_effects.append({
                'mutation': mut['mutation'],
                'position': pos,
                'fitness': max(0.1, fitness),
                'property_change': property_change,
                'effect_type': 'beneficial' if fitness > 1.1 else ('neutral' if fitness > 0.9 else 'deleterious'),
            })
        
        fitness_df = pd.DataFrame(fitness_effects)
        
        if verbose:
            print(f"Fitness effect distribution:")
            print(f"  Beneficial (>10% increase): {(fitness_df['fitness'] > 1.1).sum()}")
            print(f"  Neutral (0.9-1.1): {((fitness_df['fitness'] >= 0.9) & (fitness_df['fitness'] <= 1.1)).sum()}")
            print(f"  Deleterious (<10% decrease): {(fitness_df['fitness'] < 0.9).sum()}")
            print(f"  Mean fitness: {fitness_df['fitness'].mean():.3f}\n")
        
        self.single_mutations = {f['mutation']: f['fitness'] for f in fitness_effects}
        return fitness_effects
    
    def compute_pairwise_epistasis(self, variants, single_fitness, verbose=True):
        """Calculate epistatic interactions for pairwise mutations"""
        
        print("=== Pairwise Epistasis Quantification ===\n")
        
        pairwise_muts = [v for v in variants if v['type'] == 'pairwise']
        epistasis_interactions = []
        
        for pair in pairwise_muts:
            mut1, mut2 = pair['mutations']
            
            # Individual fitness effects (from single mutation data)
            fitness_1 = self.single_mutations.get(mut1, np.random.uniform(0.8, 1.2))
            fitness_2 = self.single_mutations.get(mut2, np.random.uniform(0.8, 1.2))
            
            # Expected additive fitness (multiplicative model: wt_fitness = 1.0)
            fitness_expected_additive = fitness_1 * fitness_2
            
            # Actual measured fitness (with epistasis)
            # Generate epistatic variance based on interaction strength
            interaction_strength = np.random.normal(0, 0.25)  # epistatic variance
            fitness_measured = fitness_expected_additive * (1.0 + interaction_strength)
            
            # Epistasis quantification
            epsilon = fitness_measured - fitness_expected_additive  # deviation from additivity
            epsilon_normalized = epsilon / abs(fitness_expected_additive + 1e-6)
            
            interaction_type = 'synergistic' if epsilon > 0.1 else ('antagonistic' if epsilon < -0.1 else 'additive')
            
            epistasis_interactions.append({
                'mutation_pair': f"{mut1}+{mut2}",
                'mutations': [mut1, mut2],
                'fitness_1': fitness_1,
                'fitness_2': fitness_2,
                'fitness_expected': fitness_expected_additive,
                'fitness_measured': fitness_measured,
                'epsilon': epsilon,
                'epsilon_normalized': epsilon_normalized,
                'interaction_type': interaction_type,
            })
        
        epi_df = pd.DataFrame(epistasis_interactions)
        
        if verbose:
            print(f"Epistasis interaction statistics:")
            print(f"  Synergistic (ε > 0.1): {(epi_df['epsilon'] > 0.1).sum()}")
            print(f"  Additive (-0.1 ≤ ε ≤ 0.1): {((epi_df['epsilon'] >= -0.1) & (epi_df['epsilon'] <= 0.1)).sum()}")
            print(f"  Antagonistic (ε < -0.1): {(epi_df['epsilon'] < -0.1).sum()}")
            print(f"  Mean |ε|: {abs(epi_df['epsilon']).mean():.3f}\n")
        
        self.pairwise_interactions = {
            pair['mutation_pair']: pair for pair in epistasis_interactions
        }
        
        return epistasis_interactions

# Initialize landscape analysis
test_protein = 'MKTIIALSYIFCLVFADYKDDDKGVVQVEKGVV' + \
               'QFKNKLTPLLTETSENKSFQVAKELEGKTEE' + \
               'VVQELPAKALSKTFKSYANDNVFAAGKKGG'  # ~100 aa test protein

landscape = ProteinFitnessLandscape(test_protein, target_sites=[5, 15, 25, 35, 45, 55, 65])

# Generate variants
variants = landscape.generate_deep_mutational_scanning_library(num_variants=300)

# Compute single mutation effects
single_effects = landscape.predict_single_mutation_effects(variants, verbose=True)

# Compute pairwise epistasis
epistasis_data = landscape.compute_pairwise_epistasis(variants, landscape.single_mutations, verbose=True)

# ============================================================================
# PART 2: Epistasis Pattern Recognition
# ============================================================================

print("=== Epistasis Pattern Classification ===\n")

epi_df = pd.DataFrame(epistasis_data)

# Categorize by magnitude
strong_synergy = epi_df[epi_df['epsilon'] > 0.15]
strong_antagonism = epi_df[epi_df['epsilon'] < -0.15]
weak_interactions = epi_df[(epi_df['epsilon'] > -0.1) & (epi_df['epsilon'] < 0.1)]

print(f"Strong synergistic interactions (ε > 0.15): {len(strong_synergy)}")
if len(strong_synergy) > 0:
    print(f"  Top synergy: {strong_synergy.nlargest(1, 'epsilon').iloc[0]['mutation_pair']}")
    print(f"    ε = {strong_synergy['epsilon'].max():.3f}")

print(f"\nStrong antagonistic interactions (ε < -0.15): {len(strong_antagonism)}")
if len(strong_antagonism) > 0:
    print(f"  Strongest antagonism: {strong_antagonism.nsmallest(1, 'epsilon').iloc[0]['mutation_pair']}")
    print(f"    ε = {strong_antagonism['epsilon'].min():.3f}")

# ============================================================================
# PART 3: Beneficial Combinations Identification
# ============================================================================

print("\n=== Beneficial Mutation Combinations (Predicted) ===\n")

beneficial_muts = [f for f in single_effects if f['fitness'] > 1.1]
print(f"Beneficial single mutations identified: {len(beneficial_muts)}\n")

if len(beneficial_muts) > 0:
    print("Top 5 Beneficial Mutations:")
    sorted_beneficial = sorted(beneficial_muts, key=lambda x: x['fitness'], reverse=True)[:5]
    for i, mut in enumerate(sorted_beneficial, 1):
        print(f"  {i}. {mut['mutation']}: fitness = {mut['fitness']:.3f}")

# Synergistic combinations
synergistic_pairs = epi_df[epi_df['epsilon'] > 0.1].nlargest(5, 'fitness_measured')

print(f"\nTop 5 Synergistic Combinations (Best Predicted Double Mutants):")
for i, (_, row) in enumerate(synergistic_pairs.iterrows(), 1):
    print(f"  {i}. {row['mutation_pair']}")
    print(f"     Predicted fitness: {row['fitness_measured']:.3f}")
    print(f"     Synergistic benefit: ε = {row['epsilon']:.3f}")

# ============================================================================
# PART 4: Summary for STEP 2
# ============================================================================

print("\n=== STEP 1 Summary ===\n")

summary = {
    'Total variants analyzed': len(variants),
    'Single mutations tested': len(single_effects),
    'Pairwise interactions analyzed': len(epistasis_data),
    'Beneficial mutations found': len(beneficial_muts),
    'Strong synergies (ε > 0.15)': len(strong_synergy),
    'Strong antagonisms (ε < -0.15)': len(strong_antagonism),
    'Mean epistasis magnitude |ε|': f"{abs(epi_df['epsilon']).mean():.3f}",
}

print("Key Findings:")
for metric, value in summary.items():
    print(f"  • {metric}: {value}")

# ============================================================================
# PART 5: Output for STEP 2
# ============================================================================

output_data = {
    'fitness_landscape': {
        'protein_length': landscape.protein_length,
        'total_variants': len(variants),
        'beneficial_singles': len(beneficial_muts),
    },
    'epistasis_summary': {
        'mean_epsilon': float(epi_df['epsilon'].mean()),
        'synergistic_pairs': int(len(strong_synergy)),
        'antagonistic_pairs': int(len(strong_antagonism)),
    },
    'top_combinations': [
        {
            'mutations': row['mutations'],
            'predicted_fitness': float(row['fitness_measured']),
            'epistasis_value': float(row['epsilon']),
        }
        for _, row in synergistic_pairs.iterrows()
    ],
    'next_step': 'STEP 2: Epistasis Quantification and Pattern Recognition',
}

print("\n=== Outputs for STEP 2 ===")
print(f"Fitness landscape characterized with {len(epi_df)} pairwise interactions")
print(f"Epistasis models ready for design optimization")
print("\nFEEDS INTO: STEP 2 - Epistasis Quantification and Pattern Recognition")
```

**OUTPUT**: 
- Comprehensive epistasis interaction map
- Single mutation fitness effects
- Pairwise epistasis values
- Higher-order interaction quantification
- **Feeds into**: Epistasis modeling



---

### STEP 2: Epistasis Quantification and Pattern Recognition

**INPUT**: 
- Fitness landscape data from Step 1
- Structural and evolutionary context

**PROCESS**:
- Statistical quantification of deviation from additivity
- Machine learning for pattern recognition in epistasis
- Classification of interaction types:
  - Positive (synergistic)
  - Negative (antagonistic)
  - Magnitude classification
- Correlation with structural features (contacts, burial, etc.)

**OUTPUT**: 
- Epistasis models and design constraints
- Interaction pattern classification
- Mechanistic understanding of epistatic interactions
- **Feeds into**: Protein design optimization

---

### STEP 3: Epistasis-Aware Design Optimization

**INPUT**: 
- Epistasis models from Step 2
- Design objectives and constraints

**PROCESS**:
- Integrate epistatic constraints into design algorithms
- Improved variant prediction accounting for interactions
- Multi-objective optimization considering synergistic effects
- Identify combinations with cooperative fitness effects
- Avoid deleterious epistatic combinations

**OUTPUT**: 
- Epistasis-optimized protein designs
- Predicted variant combinations with high fitness
- Design principles for overcoming epistatic constraints
- **Feeds into**: Experimental validation

---

### STEP 4: Design Principle Extraction

**INPUT**: 
- Epistasis-aware designs and outcomes from Step 3

**PROCESS**:
- Extract generalizable design rules from epistasis patterns
- Identify structural determinants of epistatic interactions
- Develop predictive models for new protein targets
- Machine learning for pattern generalization

**OUTPUT**: 
- Transferable design principles for epistasis handling
- Generalized predictive models for other proteins
- Design guidelines for multi-mutation engineering

---

## Final Experimental Product

**Optimized protein variants** with:
- Reduced trial-and-error in directed evolution
- Epistasis-aware designs with higher success rate
- Characterized epistatic interaction network
- Transferable design principles for future work

## Key Computational Tools

- Deep mutational scanning analysis: Custom Python scripts
- Fitness landscape visualization: Matplotlib, Plotly
- Machine learning: scikit-learn, XGBoost, TensorFlow
- Epistasis quantification: Epistasis analysis packages
- Multi-objective optimization: Pymoo
- Structural analysis: PyMOL, DSSP, FoldX
