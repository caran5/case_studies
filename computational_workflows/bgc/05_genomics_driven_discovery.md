# Workflow 5: Genomics-Driven Discovery of Microbial Natural Products

**Paper**: "Genomics-driven discovery of microbial natural products"

## Research Objective

- Leverage genome mining to overcome antibiotic resistance crisis
- Systematically discover novel natural product scaffolds
- Enable high-efficiency discovery pipeline from genomics to bioactivity

---

## Quick Reference

| Metric | Value |
|--------|-------|
| **Computational Time** | 6-10 weeks |
| **CPU Requirements** | 16-32 cores |
| **Storage** | 250 GB |
| **Languages** | Python 3.8+ |
| **Success Metric** | Discover 5-10 novel natural product scaffolds |
| **Genomes Analyzed** | 100-1,000 microbial genomes |
| **BGCs Identified** | 500-5,000 total, 50-200 priority |

---

## Computational Workflow

### STEP 1: BGC Mining and Prioritization

**INPUT**: 
- Microbial genomic sequences (shotgun or long-read)
- Knowledge of drug-resistance genes
- Prioritization criteria (novelty, therapeutic potential)

**PROCESS**: Complete executable Python code for BGC discovery and prioritization

```python
import numpy as np
import pandas as pd
from collections import defaultdict, Counter
import matplotlib.pyplot as plt
import json

print("=== Genomics-Driven Discovery - STEP 1: BGC Mining and Prioritization ===\n")

# ============================================================================
# PART 1: Simulated Microbial Genome Mining
# ============================================================================

class BGCMiner:
    def __init__(self):
        self.genomes = []
        self.detected_bgcs = []
        self.bgc_annotations = {}
    
    def load_microbial_genomes(self, n_genomes=100):
        """Load and characterize microbial genomes"""
        
        print(f"Loading {n_genomes} microbial genomes...\n")
        
        genera = [
            'Streptomyces', 'Bacillus', 'Pseudomonas', 'Burkholderia', 
            'Actinomycetes', 'Myxobacteria', 'Cyanobacteria', 'Proteobacteria'
        ]
        
        genomes = []
        for i in range(n_genomes):
            genus = np.random.choice(genera)
            genome_size_mb = np.random.normal(6, 3)  # Most bacteria 2-10 MB
            
            # BGCs more common in certain organisms
            if genus in ['Streptomyces', 'Actinomycetes', 'Myxobacteria']:
                expected_bgcs = int(genome_size_mb / 0.5)  # 1 BGC per 0.5 MB
            else:
                expected_bgcs = max(1, int(genome_size_mb / 2))  # 1 BGC per 2 MB
            
            genome = {
                'id': f"GEN_{i+1:04d}",
                'organism': f"{genus}_sp_{i+1}",
                'size_mb': genome_size_mb,
                'gc_content': np.random.normal(65, 10),
                'gene_count': int(genome_size_mb * 1000),
                'expected_bgcs': expected_bgcs,
            }
            genomes.append(genome)
        
        self.genomes = genomes
        print(f"Loaded {len(genomes)} genomes")
        print(f"  Total size: {sum(g['size_mb'] for g in genomes):.0f} MB")
        print(f"  Expected BGCs: {sum(g['expected_bgcs'] for g in genomes)}")
        
        return genomes
    
    def detect_bgcs_antismash(self, genomes, detection_rate=0.85):
        """Simulate antiSMASH BGC detection"""
        
        print(f"\n=== BGC Detection with antiSMASH ===\n")
        
        bgcs_detected = []
        bgc_id = 0
        
        for genome in genomes:
            # Simulate detection (not all BGCs detected)
            n_detected = int(genome['expected_bgcs'] * detection_rate)
            
            for bgc_num in range(n_detected):
                bgc_id += 1
                
                # BGC characteristics
                bgc_size = np.random.randint(15, 100)  # kilobases
                gene_count = np.random.randint(5, 25)
                
                # BGC type (rough distribution in nature)
                bgc_types = ['polyketide', 'nonribosomal_peptide', 'terpene', 
                            'ribosomal_peptide', 'hybrid', 'fatty_acid']
                bgc_type = np.random.choice(bgc_types, p=[0.25, 0.25, 0.15, 0.15, 0.15, 0.05])
                
                # Core enzyme completeness (some incomplete BGCs)
                core_enzymes_complete = np.random.random() > 0.2  # 80% complete
                
                bgc = {
                    'id': f"BGC_{bgc_id:06d}",
                    'genome_id': genome['id'],
                    'organism': genome['organism'],
                    'size_kb': bgc_size,
                    'gene_count': gene_count,
                    'type': bgc_type,
                    'core_enzymes_complete': core_enzymes_complete,
                    'gc_content': np.random.normal(65, 5),
                    'detection_score': np.random.uniform(0.5, 1.0),
                }
                bgcs_detected.append(bgc)
        
        self.detected_bgcs = bgcs_detected
        
        print(f"Detected {len(bgcs_detected)} total BGCs")
        print(f"  Detection rate: {len(bgcs_detected) / sum(g['expected_bgcs'] for g in genomes) * 100:.1f}%")
        print(f"\nBGC Type Distribution:")
        bgc_types = Counter(b['type'] for b in bgcs_detected)
        for bgc_type, count in bgc_types.most_common():
            print(f"  {bgc_type}: {count} ({count/len(bgcs_detected)*100:.1f}%)")
        
        return bgcs_detected

    def assess_bgc_novelty(self, bgcs, reference_bgc_count=5000):
        """Assess novelty of detected BGCs against known BGC databases"""
        
        print(f"\n=== BGC Novelty Assessment ===\n")
        
        # Simulate sequence similarity to known BGCs
        # Novel BGCs have low similarity to known BGCs
        
        novelty_scores = []
        
        for bgc in bgcs:
            # Simulate comparison to known database
            # Novel BGCs: avg similarity 20-40%
            # Known BGCs: avg similarity 60-90%
            
            is_likely_known = np.random.random() < 0.3  # ~30% are known types
            
            if is_likely_known:
                max_similarity = np.random.uniform(0.6, 0.95)
                novel_score = 1.0 - max_similarity  # Lower score = more known
            else:
                max_similarity = np.random.uniform(0.15, 0.45)
                novel_score = 1.0 - max_similarity  # Higher score = more novel
            
            # Novelty bonus for rare organism
            organism_frequency = np.random.uniform(0, 1)
            organism_novelty_bonus = (1.0 - organism_frequency) * 0.2
            
            final_novelty_score = min(1.0, novel_score + organism_novelty_bonus)
            
            bgc['max_similarity_to_known'] = max_similarity
            bgc['novelty_score'] = final_novelty_score
            bgc['is_likely_novel'] = final_novelty_score > 0.6
            
            novelty_scores.append(final_novelty_score)
        
        # Statistics
        novelty_arr = np.array(novelty_scores)
        novel_bgcs = sum(1 for score in novelty_scores if score > 0.6)
        
        print(f"Novelty assessment results:")
        print(f"  Mean novelty score: {novelty_arr.mean():.3f}")
        print(f"  Likely novel BGCs (score > 0.6): {novel_bgcs} ({novel_bgcs/len(bgcs)*100:.1f}%)")
        print(f"  Completely new scaffolds (score > 0.8): {sum(1 for s in novelty_scores if s > 0.8)}")
        
        return novelty_scores

    def predict_bioactivity_potential(self, bgcs):
        """Predict bioactivity potential based on BGC characteristics"""
        
        print(f"\n=== Bioactivity Potential Prediction ===\n")
        
        bioactivity_predictions = []
        
        # Bioactivity indicators based on BGC type and completeness
        bioactivity_prob = {
            'polyketide': 0.65,
            'nonribosomal_peptide': 0.75,
            'terpene': 0.45,
            'ribosomal_peptide': 0.70,
            'hybrid': 0.80,
            'fatty_acid': 0.30,
        }
        
        for bgc in bgcs:
            bgc_type = bgc['type']
            
            # Base probability from BGC type
            base_prob = bioactivity_prob.get(bgc_type, 0.5)
            
            # Adjust based on completeness
            if bgc['core_enzymes_complete']:
                completeness_factor = 1.1
            else:
                completeness_factor = 0.7
            
            # Adjust based on novelty
            novelty_factor = 0.8 + 0.4 * bgc['novelty_score']
            
            # Final prediction
            bioactivity_prob_final = min(0.95, base_prob * completeness_factor * novelty_factor)
            
            bgc['bioactivity_probability'] = bioactivity_prob_final
            bgc['likely_active'] = bioactivity_prob_final > 0.5
            
            bioactivity_predictions.append({
                'bgc_id': bgc['id'],
                'type': bgc_type,
                'completeness': bgc['core_enzymes_complete'],
                'novelty': bgc['novelty_score'],
                'bioactivity_prob': bioactivity_prob_final,
            })
        
        pred_df = pd.DataFrame(bioactivity_predictions)
        
        print(f"BGCs with high bioactivity potential (prob > 0.5): {(pred_df['bioactivity_prob'] > 0.5).sum()}")
        print(f"Mean bioactivity probability: {pred_df['bioactivity_prob'].mean():.3f}")
        
        return bioactivity_predictions

# ============================================================================
# PART 2: BGC Prioritization Pipeline
# ============================================================================

class BGCPrioritizer:
    def __init__(self, bgcs):
        self.bgcs = bgcs
        self.priority_scores = []
    
    def calculate_priority_scores(self, novelty_weight=0.4, bioactivity_weight=0.4, 
                                  completeness_weight=0.2):
        """Calculate composite priority score for each BGC"""
        
        print(f"\n=== BGC Prioritization ===\n")
        
        priorities = []
        
        for bgc in self.bgcs:
            # Normalize scores to 0-1 range
            novelty_norm = bgc['novelty_score']
            bioactivity_norm = bgc['bioactivity_probability']
            completeness_norm = 1.0 if bgc['core_enzymes_complete'] else 0.6
            
            # Weighted composite score
            priority_score = (novelty_weight * novelty_norm + 
                            bioactivity_weight * bioactivity_norm + 
                            completeness_weight * completeness_norm)
            
            bgc['priority_score'] = priority_score
            bgc['priority_rank'] = 0  # Will be assigned after sorting
            
            priorities.append({
                'bgc_id': bgc['id'],
                'organism': bgc['organism'],
                'type': bgc['type'],
                'novelty': bgc['novelty_score'],
                'bioactivity': bgc['bioactivity_probability'],
                'completeness': 1.0 if bgc['core_enzymes_complete'] else 0.6,
                'priority_score': priority_score,
            })
        
        # Rank by priority score
        self.bgcs.sort(key=lambda x: x['priority_score'], reverse=True)
        for rank, bgc in enumerate(self.bgcs, 1):
            bgc['priority_rank'] = rank
        
        priorities_df = pd.DataFrame(priorities)
        priorities_df.sort_values('priority_score', ascending=False, inplace=True)
        
        return priorities_df
    
    def select_priority_bgcs(self, n_top=50):
        """Select top priority BGCs for downstream expression/validation"""
        
        print(f"Selected top {n_top} priority BGCs for expression\n")
        
        top_bgcs = self.bgcs[:n_top]
        
        print("Top 10 Priority BGCs:")
        print("Rank | BGC_ID    | Organism           | Type     | Novelty | Bioactivity | Priority")
        print("-" * 85)
        
        for i, bgc in enumerate(top_bgcs[:10], 1):
            print(f"{i:4d} | {bgc['id'][:9]:>9} | {bgc['organism'][:18]:18} | {bgc['type'][:8]:>8} | "
                  f"{bgc['novelty_score']:7.3f} | {bgc['bioactivity_probability']:11.3f} | {bgc['priority_score']:.3f}")
        
        return top_bgcs

# ============================================================================
# PART 3: Run Complete Workflow
# ============================================================================

print("Step 1: Load Microbial Genomes")
print("=" * 60)

miner = BGCMiner()
genomes = miner.load_microbial_genomes(n_genomes=100)

print("\n\nStep 2: Detect BGCs")
print("=" * 60)

bgcs = miner.detect_bgcs_antismash(genomes, detection_rate=0.85)

print("\n\nStep 3: Assess Novelty")
print("=" * 60)

novelty_scores = miner.assess_bgc_novelty(bgcs)

print("\n\nStep 4: Predict Bioactivity")
print("=" * 60)

bioactivity = miner.predict_bioactivity_potential(bgcs)

print("\n\nStep 5: Prioritize BGCs")
print("=" * 60)

prioritizer = BGCPrioritizer(miner.detected_bgcs)
priorities_df = prioritizer.calculate_priority_scores(
    novelty_weight=0.4, 
    bioactivity_weight=0.4, 
    completeness_weight=0.2
)

top_priority_bgcs = prioritizer.select_priority_bgcs(n_top=50)

# ============================================================================
# PART 6: Summary for STEP 2
# ============================================================================

print("\n\n=== STEP 1 Summary ===\n")

summary = {
    'Genomes analyzed': len(genomes),
    'Total BGCs detected': len(bgcs),
    'Likely novel BGCs': sum(1 for b in bgcs if b['is_likely_novel']),
    'Bioactive potential BGCs': sum(1 for b in bgcs if b['likely_active']),
    'Complete BGCs': sum(1 for b in bgcs if b['core_enzymes_complete']),
    'Top priority BGCs selected': len(top_priority_bgcs),
}

print("Key Findings:")
for metric, value in summary.items():
    print(f"  • {metric}: {value}")

# ============================================================================
# PART 7: Output for STEP 2
# ============================================================================

output_summary = {
    'workflow_summary': {
        'genomes_analyzed': len(genomes),
        'bgcs_detected': len(bgcs),
        'priority_bgcs_selected': len(top_priority_bgcs),
    },
    'priority_bgcs': [
        {
            'id': b['id'],
            'organism': b['organism'],
            'type': b['type'],
            'priority_score': float(b['priority_score']),
            'novelty': float(b['novelty_score']),
            'bioactivity_probability': float(b['bioactivity_probability']),
        }
        for b in top_priority_bgcs[:20]
    ],
    'next_step': 'STEP 2: Expression Strategy Selection',
}

print("\n=== Outputs for STEP 2 ===")
print(f"Identified {len(top_priority_bgcs)} priority BGCs for expression")
print(f"Likely novel discoveries: {sum(1 for b in top_priority_bgcs if b['is_likely_novel'])}")
print("\nFEEDS INTO: STEP 2 - Expression Strategy Selection")
```

**OUTPUT**: 
- Prioritized list of novel/interesting BGCs
- Novelty scores and structural annotations
- **Feeds into**: Expression strategy selection



---

### STEP 2: Expression Strategy Selection

**INPUT**: 
- Priority BGCs from Step 1
- Available heterologous expression systems

**PROCESS**:
- Computational evaluation of heterologous vs. native expression
- Host compatibility predictions
- Metabolic pathway analysis
- Expression optimization strategy selection
- Pathway engineering requirements assessment

**OUTPUT**: 
- Selected expression approach with optimized parameters
- Host system specifications
- Pathway modifications if needed
- **Feeds into**: Pathway engineering

---

### STEP 3: Pathway Engineering for Production Optimization

**INPUT**: 
- Expression system from Step 2
- BGC structural information

**PROCESS**:
- Predictive modeling of rate-limiting steps
- Enzyme expression level optimization
- Precursor availability assessment and improvement
- Metabolic flux balance analysis
- Bottleneck identification and resolution strategies

**OUTPUT**: 
- Optimized pathway engineering specifications
- Expected production level improvements
- **Feeds into**: Regulatory manipulation or direct production

---

### STEP 4: Global Regulatory Manipulation Strategy

**INPUT**: 
- Optimized pathway from Step 3
- Target production level specifications

**PROCESS**:
- Computational design of regulatory modifications
- Prediction of pleiotropic effects
- Co-culture optimization modeling
- Environmental induction strategies
- Global gene expression modeling

**OUTPUT**: 
- Regulatory modification specifications
- Co-culture parameters
- Environmental conditions optimization
- **Feeds into**: Production and bioactivity screening

---

### STEP 5: Bioactivity Assessment and Drug Development

**INPUT**: 
- Produced compounds from optimized system
- Target bioactivity profiles

**PROCESS**:
- High-throughput bioactivity screening
- Mechanism of action prediction
- Structure-activity relationship analysis
- Lead optimization recommendations
- Clinical relevance assessment

**OUTPUT**: 
- Novel natural products with improved production levels
- New drug candidates addressing resistance
- Development recommendations

---

## Final Experimental Product

**Novel bioactive compounds** with:
- Genome-driven discovery approach
- Optimized production systems
- Characterized bioactivity
- Ready for preclinical/clinical development

## Key Computational Tools

- BGC mining: antiSMASH, MINER, ClusterMine
- Sequence analysis: BLAST, HMMsearch, FASTA
- Genome assembly: SPAdes, Velvet, DISCOVAR
- Metabolic modeling: COBRA, Gurobi, CPLEX
- Metabolic reconstruction: KBase, RAST
- Flux balance analysis: COBRApy, PSAMM
- Expression analysis: RBS Calculator, Promoter predictors
- Bioactivity prediction: Activity prediction ML models
- Co-culture modeling: Community simulation tools
