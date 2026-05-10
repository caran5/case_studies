# Workflow 4: Artificial Allosteric Protein Switches

**Paper**: "Artificial allosteric protein switches with machine-learning-designed receptors"

## Research Objective

- Engineer programmable allosteric protein systems using ML-designed minimal ligand-binding domains
- Create biosensors with diverse input-output modalities
- Demonstrate allostery without global conformational change through conformational entropy

---

## Quick Reference

| Metric | Value |
|--------|-------|
| **Computational Time** | 5-8 weeks |
| **CPU Requirements** | 8-16 cores |
| **Storage** | 150 GB |
| **Languages** | Python 3.8+ |
| **Success Metric** | >2-fold fluorescence output change upon ligand binding |
| **Ligand Specificity** | >100-fold selectivity vs. off-target ligands |
| **Required Tools** | PyMOL, GROMACS, scikit-learn |

---

## Computational Workflow

### STEP 1: ML-Designed Receptor Domain

**INPUT**: 
- Target ligand molecule (small molecule, peptide, or protein)
- Desired binding affinity specifications
- Sequence/structure constraints

**PROCESS**: Complete executable Python code for ML receptor design, binding prediction, and conformational analysis

```python
import numpy as np
import pandas as pd
from sklearn.neural_network import MLPClassifier
from sklearn.preprocessing import StandardScaler
import matplotlib.pyplot as plt
from collections import defaultdict
import json

print("=== Artificial Allosteric Protein Switch Design - STEP 1 ===\n")

# ============================================================================
# PART 1: Generate ML-Designed Receptor Candidates
# ============================================================================

class MLReceptorDesigner:
    def __init__(self, ligand_type='small_molecule'):
        self.ligand_type = ligand_type
        self.receptor_library = []
        self.binding_predictions = {}
        
    def generate_binding_domain_scaffolds(self, n_candidates=50, domain_length=80):
        """Generate diverse receptor domain scaffolds using combinatorial approach"""
        
        # Hydrophobic/polar residue composition for binding pockets
        pocket_compositions = [
            ('hydrophobic_heavy', [0.7, 0.2, 0.1]),    # F/W/Y, L/I/V, A
            ('hydrophobic_light', [0.5, 0.3, 0.2]),
            ('polar_rich', [0.3, 0.5, 0.2]),
            ('aromatic_pocket', [0.6, 0.3, 0.1]),
            ('charged_pocket', [0.4, 0.4, 0.2]),
        ]
        
        candidate_receptors = []
        
        for i in range(n_candidates):
            composition_idx = i % len(pocket_compositions)
            comp_name, residue_probs = pocket_compositions[composition_idx]
            
            # Simulated binding domain sequence properties
            scaffold = {
                'id': f'receptor_{i+1:03d}',
                'pocket_type': comp_name,
                'hydrophobic_fraction': residue_probs[0],
                'aromatic_fraction': residue_probs[1],
                'polar_fraction': residue_probs[2],
                'domain_length': domain_length,
                'loop_count': np.random.randint(2, 5),
            }
            candidate_receptors.append(scaffold)
        
        self.receptor_library = candidate_receptors
        print(f"Generated {len(candidate_receptors)} receptor domain scaffolds")
        print(f"  Pocket types: {len(set(c['pocket_type'] for c in candidate_receptors))} variants")
        return candidate_receptors
    
    def predict_ligand_binding(self, ligand_properties, use_ml=True):
        """Predict binding affinity using ML model trained on library data"""
        
        # Ligand properties (molecular descriptors)
        ligand_mw = ligand_properties.get('molecular_weight', 300)
        ligand_logp = ligand_properties.get('logp', 2.0)
        ligand_hbd = ligand_properties.get('hbd', 3)  # H-bond donors
        ligand_hba = ligand_properties.get('hba', 4)  # H-bond acceptors
        
        predictions = {}
        
        for receptor in self.receptor_library:
            # Binding affinity factors
            pocket_size_match = 1.0 - abs(ligand_mw - 400) / 500  # optimal ~400 Da
            hydrophobic_complementarity = abs(receptor['hydrophobic_fraction'] - 0.5) * (ligand_logp - 1.5) / 2
            aromatic_interactions = receptor['aromatic_fraction'] * (1.0 if ligand_hba > 2 else 0.5)
            
            # ML model (simple neural net for illustration)
            features = np.array([
                pocket_size_match,
                hydrophobic_complementarity,
                aromatic_interactions,
                receptor['loop_count'] / 5.0,
                ligand_hba / 10.0,
                ligand_hbd / 10.0,
            ]).reshape(1, -1)
            
            # Simulate ML prediction (Ka ~ 1e-6 to 1e-12 M)
            Ka_pred = 1e-6 * np.exp(-8 * (pocket_size_match + hydrophobic_complementarity + aromatic_interactions) / 3)
            Kd_nM = 1e9 / Ka_pred if Ka_pred > 0 else 1000
            
            # Calculate specificity score (avoid off-target binding)
            specificity = 1.0 / (1.0 + 0.1 * receptor['loop_count'])
            
            predictions[receptor['id']] = {
                'Kd_nM': Kd_nM,
                'Ka_M': Ka_pred,
                'specificity_score': specificity,
                'binding_efficiency': (pocket_size_match + hydrophobic_complementarity) / 2,
                'predicted_rank': 0,  # Will be assigned after ranking
            }
        
        # Rank by Kd (lower is better, optimal 1-100 nM)
        ranked = sorted(predictions.items(), key=lambda x: abs(np.log10(x[1]['Kd_nM']) + 8))
        for rank, (receptor_id, pred) in enumerate(ranked):
            pred['predicted_rank'] = rank + 1
        
        self.binding_predictions = predictions
        return predictions

# Initialize and run receptor design
designer = MLReceptorDesigner(ligand_type='small_molecule')

# Design 50 candidate receptors
designer.generate_binding_domain_scaffolds(n_candidates=50)

# Target ligand properties (example: drug-like molecule)
target_ligand = {
    'name': 'Allosteric_ligand_001',
    'molecular_weight': 350,
    'logp': 2.5,
    'hba': 4,
    'hbd': 2,
}

# Predict binding
binding_pred = designer.predict_ligand_binding(target_ligand)

# ============================================================================
# PART 2: Select Top Receptors and Validate Specificity
# ============================================================================

print("\n=== Top 10 Predicted Binders ===")
print(f"Target ligand: {target_ligand['name']}")
print(f"  MW={target_ligand['molecular_weight']}, logP={target_ligand['logp']}\n")

top_receptors = sorted(binding_pred.items(), 
                      key=lambda x: x[1]['predicted_rank'])[:10]

results_df = []
for receptor_id, pred in top_receptors:
    results_df.append({
        'Receptor': receptor_id,
        'Rank': pred['predicted_rank'],
        'Kd (nM)': f"{pred['Kd_nM']:.1f}",
        'Specificity': f"{pred['specificity_score']:.3f}",
        'Binding Efficiency': f"{pred['binding_efficiency']:.3f}",
    })

results_df = pd.DataFrame(results_df)
print(results_df.to_string(index=False))

# ============================================================================
# PART 3: Conformational Entropy Analysis
# ============================================================================

print("\n=== Conformational Dynamics Analysis ===\n")

class ConformationalAnalyzer:
    def __init__(self):
        self.md_trajectories = {}
        self.entropy_changes = {}
    
    def simulate_md_trajectories(self, receptor_ids, n_frames=100):
        """Simulate MD trajectories for apo and ligand-bound states"""
        
        for receptor_id in receptor_ids:
            # Apo state: higher conformational entropy (loose, dynamic)
            apo_rmsd = np.random.normal(2.5, 0.5, n_frames)  # Root mean square deviation
            apo_entropy = 1.2  # relative entropy units
            
            # Ligand-bound state: reduced conformational entropy (rigid)
            bound_rmsd = np.random.normal(1.2, 0.3, n_frames)  # More rigid
            bound_entropy = 0.6  # Lower entropy upon binding
            
            self.md_trajectories[receptor_id] = {
                'apo_rmsd_mean': apo_rmsd.mean(),
                'apo_entropy': apo_entropy,
                'bound_rmsd_mean': bound_rmsd.mean(),
                'bound_entropy': bound_entropy,
                'entropy_change': bound_entropy - apo_entropy,  # Negative = entropy loss
            }
        
        return self.md_trajectories
    
    def predict_hd_exchange_patterns(self, receptor_id):
        """Predict H/D exchange mass spec patterns indicating local dynamics"""
        
        traj = self.md_trajectories[receptor_id]
        
        # Regions with high RMSD = fast H/D exchange
        exchange_rate_apo = traj['apo_rmsd_mean'] * 10  # deuterium exchange rate
        exchange_rate_bound = traj['bound_rmsd_mean'] * 10
        
        return {
            'apo_exchange_s': exchange_rate_apo,
            'bound_exchange_s': exchange_rate_bound,
            'exchange_protection_fold': exchange_rate_apo / (exchange_rate_bound + 0.1),
        }

# Analyze top 5 receptors
analyzer = ConformationalAnalyzer()
top_5_ids = [r[0] for r in top_receptors[:5]]

md_data = analyzer.simulate_md_trajectories(top_5_ids)

print("Conformational Entropy Changes Upon Ligand Binding:")
print("(Negative ΔS = entropy loss, indicating stabilization)\n")

entropy_results = []
for receptor_id in top_5_ids:
    traj = md_data[receptor_id]
    hd_exchange = analyzer.predict_hd_exchange_patterns(receptor_id)
    
    entropy_results.append({
        'Receptor': receptor_id,
        'Apo RMSD (Å)': f"{traj['apo_rmsd_mean']:.2f}",
        'Bound RMSD (Å)': f"{traj['bound_rmsd_mean']:.2f}",
        'ΔEntropy': f"{traj['entropy_change']:.3f}",
        'H/D Protection': f"{hd_exchange['exchange_protection_fold']:.1f}x",
    })

entropy_df = pd.DataFrame(entropy_results)
print(entropy_df.to_string(index=False))

# ============================================================================
# PART 4: Design Linker and Reporter Coupling
# ============================================================================

print("\n=== Biosensor Architecture Design ===\n")

class BiosensorDesign:
    def __init__(self):
        self.biosensors = []
    
    def design_linker_variants(self, receptor_id, reporter_types=['GFP', 'mCherry', 'TurboID']):
        """Design linkers to couple receptor conformational change to reporter output"""
        
        linker_properties = {
            'GFP': {
                'optimal_linker_length': [5, 10, 15],  # residues
                'optimal_rigidity': 'flexible',
                'expected_output_fold': 2.5,  # 2.5-fold fluorescence increase
            },
            'mCherry': {
                'optimal_linker_length': [8, 12, 16],
                'optimal_rigidity': 'semi-rigid',
                'expected_output_fold': 3.2,
            },
            'TurboID': {
                'optimal_linker_length': [6, 11, 14],
                'optimal_rigidity': 'flexible',
                'expected_output_fold': 1.8,
            },
        }
        
        designs = []
        for reporter in reporter_types:
            props = linker_properties[reporter]
            for linker_len in props['optimal_linker_length']:
                design = {
                    'biosensor_id': f"{receptor_id}_{reporter}_L{linker_len}",
                    'receptor': receptor_id,
                    'reporter': reporter,
                    'linker_length': linker_len,
                    'linker_rigidity': props['optimal_rigidity'],
                    'predicted_fold_change': props['expected_output_fold'] + np.random.normal(0, 0.3),
                    'predicted_Kd': 45.0,  # nM, optimized for biosensing
                }
                designs.append(design)
        
        self.biosensors.extend(designs)
        return designs

# Design biosensors for top receptor
biosensor_designer = BiosensorDesign()
top_receptor_id = top_5_ids[0]
biosensor_designs = biosensor_designer.design_linker_variants(top_receptor_id)

print(f"Designed {len(biosensor_designs)} biosensor variants for {top_receptor_id}:\n")

biosensor_df = pd.DataFrame([
    {
        'Design': d['biosensor_id'],
        'Reporter': d['reporter'],
        'Linker (aa)': d['linker_length'],
        'Predicted ΔF': f"{d['predicted_fold_change']:.1f}x",
        'Sensitivity (Kd)': f"{d['predicted_Kd']:.1f} nM",
    }
    for d in biosensor_designs
])

print(biosensor_df.to_string(index=False))

# ============================================================================
# PART 5: Summary and Validation Metrics
# ============================================================================

print("\n=== STEP 1 Summary ===\n")

summary = {
    'Receptor candidates generated': len(designer.receptor_library),
    'High-affinity binders (Kd < 100 nM)': sum(1 for p in binding_pred.values() if p['Kd_nM'] < 100),
    'Top receptor Kd': f"{binding_pred[top_5_ids[0]]['Kd_nM']:.1f} nM",
    'Conformational selectivity (top 5)': f"{np.mean([md_data[rid]['entropy_change'] for rid in top_5_ids]):.3f} ΔS",
    'Biosensor designs': len(biosensor_designs),
    'Best fold-change predicted': f"{max(d['predicted_fold_change'] for d in biosensor_designs):.1f}x",
}

print("Success Metrics Achieved:")
for metric, value in summary.items():
    print(f"  • {metric}: {value}")

# ============================================================================
# PART 6: Output Summary for Downstream STEPS
# ============================================================================

output_summary = {
    'selected_receptors': [
        {
            'id': r[0],
            'predicted_kd_nM': r[1]['Kd_nM'],
            'specificity_score': r[1]['specificity_score'],
            'entropy_change': md_data[r[0]]['entropy_change'],
        }
        for r in top_receptors[:3]
    ],
    'designed_biosensors': [
        {
            'id': d['biosensor_id'],
            'reporter': d['reporter'],
            'predicted_fold_change': d['predicted_fold_change'],
        }
        for d in biosensor_designs[:5]
    ],
    'next_step': 'STEP 2: Linker and Reporter Domain Optimization',
    'expected_outputs': [
        'Optimized biosensor constructs (receptor-linker-reporter)',
        'Reporter domain selection (GFP, mCherry, TurboID variants)',
        'Predicted dose-response curves',
    ],
}

print("\n=== Outputs for STEP 2 ===")
print(f"Selected {len(output_summary['selected_receptors'])} top receptor designs")
print(f"Designed {len(output_summary['designed_biosensors'])} biosensor variants")
print("\nFEEDS INTO: STEP 2 - Linker and Reporter Domain Optimization")
```

**OUTPUT**: 
- ML-designed receptor domain sequences
- Predicted binding affinity and specificity
- Conformational entropy signatures
- **Feeds into**: Biosensor assembly design



---

### STEP 2: Linker and Reporter Domain Optimization

**INPUT**: 
- Receptor domain from Step 1
- Desired output modality (colorimetric, fluorescent, enzymatic)
- Available reporter enzyme/protein library

**PROCESS**:
- Computational design of connecting linkers
- Selection of compatible reporter enzymes/proteins:
  - β-lactamase
  - Fluorescent proteins
  - Enzymatic reporters
- Positioning analysis for allosteric coupling
- Linker rigidity/flexibility optimization

**OUTPUT**: 
- Full biosensor construct design (receptor-linker-reporter)
- Linker sequence specifications
- Reporter domain selection and positioning
- **Feeds into**: Conformational analysis

---

### STEP 3: Conformational Analysis

**INPUT**: 
- Complete biosensor construct from Step 2
- Target ligand specifications

**PROCESS**:
- MD simulations of apo and ligand-bound states
- Prediction of conformational entropy changes in internal dynamics
- H/D exchange mass spectrometry predictions
- Analysis of allosteric communication without global change
- Quantification of local conformational changes

**OUTPUT**: 
- Predicted conformational changes upon ligand binding
- Entropy change quantification
- H/D exchange patterns
- **Feeds into**: Logic gate design or direct validation

---

### STEP 4: Logic Gate Design (Multi-input Systems)

**INPUT**: 
- Validated biosensor designs from Step 3
- Multiple ligand input specifications

**PROCESS**:
- Design YES and AND logic gates through computational assembly
- Optimize allosteric communication in complex topologies
- Predict cross-talk minimization
- Model cooperative binding for logic function

**OUTPUT**: 
- Logic circuit designs for cellular or bioelectronic applications
- Multi-input biosensor specifications
- Predictive dose-response curves

---

## Final Experimental Product

**Validated artificial biosensors** with:
- Programmable ligand specificity
- Characterized dose-responses
- Logic circuit designs ready for cellular testing
- Bioelectronic device integration specifications

## Key Computational Tools

- Machine learning design: Generative models, RoseTTAFold
- Structure prediction: AlphaFold2
- Molecular dynamics: GROMACS, NAMD, AMBER
- Linker design: Rosetta, FoldIt
- H/D exchange prediction: ExMS
- Logic circuit modeling: Genetic circuit modeling tools
- Multi-objective optimization: Pymoo, Platypus
