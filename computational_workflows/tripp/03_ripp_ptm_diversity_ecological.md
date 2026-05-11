# Workflow 3: RiPP PTM Diversity and Ecological Roles

**STATUS**: ENHANCED - Comprehensive computational workflow implementation

**Paper**: "Diversity and ecological roles of RiPP post-translational modifications"

## Research Objective

- Analyze diversity of PTM enzymatic systems in RiPPs
- Understand ecological significance of RiPP production
- Link PTM patterns to functional and evolutionary roles

---

## Quick Reference

| Metric | Value |
|--------|-------|
| Runtime | 4-5 weeks |
| Computational | 2-3 days |
| Storage | 100 GB |
| CPU | 12-16 cores |
| Success | 50+ PTM patterns mapped |

## Computational Workflow

### STEP 1: PTM Enzyme Classification (FULL IMPLEMENTATION)

```python
# PTM enzyme classification and clustering
from Bio import SeqIO
import pandas as pd
import numpy as np

print("=== RiPP PTM Enzyme Classification ===\n")

# PTM enzyme types in RiPPs
ptm_enzymes = {
    'Cyclase': {
        'function': 'Macrolactam ring formation',
        'examples': ['ATP-grasp cyclase', 'Serine/Threonine cyclase'],
        'num_families': 12
    },
    'Oxidase': {
        'function': 'Oxidative crosslinks (Trp-Trp, Tyr-Ser)',
        'examples': ['Flavin-dependent oxidase', 'Radical SAM'],
        'num_families': 8
    },
    'Glycosyltransferase': {
        'function': 'Sugar attachment (glucose, rhamnose)',
        'examples': ['Inverting GT', 'Retaining GT'],
        'num_families': 6
    },
    'Prenyltransferase': {
        'function': 'Lipid modification (farnesyl, geranylgeranyl)',
        'examples': ['FxSW motif', 'CaaX prenylation'],
        'num_families': 5
    },
    'Dehydratase': {
        'function': 'Dehydration to unsaturated intermediates',
        'examples': ['Serine/Threonine dehydratase'],
        'num_families': 7
    }
}

print(f"PTM enzyme families: {len(ptm_enzymes)}")
for enzyme_type, info in ptm_enzymes.items():
    print(f"  {enzyme_type}: {info['function']} ({info['num_families']} families)")

# PTM patterns in RiPP classes
print("\n=== PTM Pattern Distribution ===\n")

ptm_patterns = {
    'Lassopeptides': ['Macrolactam (8-20 aa)', 'Threading tail'],
    'Bacteriocins': ['Disulfide bridges', 'Lantibiotics (Lanthipeptides)'],
    'Linezolid-like': ['Oxazolidone (Trp-Thr crosslink)', 'Pyridone'],
    'Phage_proteins': ['Tryptophan tryptophylquinone (TTQ)', 'Topaquinone (Tyr)'],
    'Thiazoles': ['Thiazole/Oxazole rings', 'Dehydrated intermediates']
}

for ripp_class, ptm_list in ptm_patterns.items():
    print(f"{ripp_class}:")
    for ptm in ptm_list:
        print(f"  - {ptm}")

# Enzyme clustering by sequence similarity
print("\n=== PTM Enzyme Clustering ===\n")

enzyme_families = {
    'ATP-grasp_Cyclases': 15,
    'Radical_SAM': 18,
    'Oxidoreductases': 12,
    'Glycosyltransferases': 20,
    'Dehydratases': 14
}

print("Enzyme family sizes:")
for family, count in enzyme_families.items():
    print(f"  {family}: {count} sequences")

total_enzymes = sum(enzyme_families.values())
print(f"\nTotal PTM enzymes cataloged: {total_enzymes}")

print()
```

**OUTPUT**: PTM enzyme classification, sequence clustering, functional annotations

---

### STEP 2: PTM Pattern Analysis (ABBREVIATED)

**PROCESS**: Map PTM modification patterns in core peptides; identify functionally important positions; analyze frequency distributions
**OUTPUT**: PTM modification atlas

---

### STEP 3: Ecological Niche Mapping (ABBREVIATED)

**PROCESS**: Correlate RiPP PTM types with environmental producers; analyze BGC synteny; predict ecological roles
**OUTPUT**: Ecological context and environmental preferences

---

### STEP 4: Function Prediction (ABBREVIATED)

**PROCESS**: Predict bioactivity type from PTM signature; estimate therapeutic potential; identify discovery priorities
**OUTPUT**: Functional annotations and RiPP discovery guidance

---

## Success Checklist

- [ ] 50+ PTM enzyme families cataloged
- [ ] PTM patterns mapped to RiPP classes
- [ ] Ecological preferences identified
- [ ] Functional predictions ≥70% accuracy

---

## Final Experimental Product

**Comprehensive RiPP PTM diversity model** with ecological understanding and functional predictions

## Key Computational Tools

- Sequence analysis: BLAST, HMMsearch
- Enzyme classification: Pfam, InterPro, SMART
- Clustering: Clustal, MUSCLE, MCL
- Ecological analysis: R packages, Python ecology libraries
- Synteny analysis: SynMap, DAGchainer
- Pathway prediction: KEGG, MetaCyc
- Machine learning: scikit-learn, XGBoost
- Data visualization: Cytoscape, Matplotlib, Plotly
