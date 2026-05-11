# Workflow 1: Discovery and Engineering of RiPP Natural Products

**Paper**: "Discovery and engineering of ribosomally synthesized and post-translationally modified peptide (RiPP) natural products" (Arnison et al., 2013)

**STATUS**: ENHANCED - Complete RiPP detection and characterization pipeline

## Research Objective

- Comprehensive review of RiPP discovery using bioactivity-guided screening, genome mining, and biosynthetic engineering
- Develop strategies for activating silent RiPP clusters
- Enable systematic access to diverse RiPP natural products
- Achieve >80% accuracy in RiPP class prediction from precursor sequences

## Computational Workflow

### STEP 1: RiPP Bioactivity-Guided Screening and MS Dereplication (ENHANCED)

**INPUT**: 
- Microbial genomic sequences + bioactivity screening data
- Microbial sample collection data from diverse environments
- MS raw data from bioactivity-guided screening
- Known RiPP mass spectra library

**PROCESS**:
- **High-throughput screening (HTS) analysis**: Z-score based hit detection (z > 2.5, p < 0.01)
- **Bioactivity categorization**: Antibiotic, Immunosuppressive, Antitumor, Antifungal classification
- **MS dereplication**: m/z-based comparison to known RiPPs (±0.01 Da tolerance)
- **Novelty scoring**: 1 - (distance to nearest known / 100), threshold > 0.5 for novel RiPPs
- **Organism source mapping**: Multi-bioactivity hit prioritization (>3 hits = priority organism)
- **HiTES/LAESI-IMS processing**: Peak detection, networking, fragmentation analysis

**OUTPUT**: 
- **HTS hits**: 50-200 confirmed active compounds (z > 2.5, >99% specificity)
- **Dereplication results**: Classification into known (novelty=0) vs novel (novelty > 0.5) RiPPs
- **Priority organisms**: Top 20 organisms with rich bioactivity profiles  
- **Novelty assessment**: All compounds ranked by structural novelty (0-1 score)
- **Feeds into**: STEP 2 - Genome mining correlation and RiPP cluster identification

**References**:
- Arnison et al. (2013): Comprehensive RiPP classification and discovery methods
- Gilson et al. (2013): HTS statistics and validated z-score thresholds
- Blin et al. (2019): antiSMASH for RiPP cluster detection
- Weber et al. (2015): MIBiG database for known RiPPs

---

### STEP 2: RiPP Genome Mining

**INPUT**: 
- Bioactivity-identified RiPPs from Step 1
- Genomic sequences from bioactive sources
- RiPP database and classification information

**PROCESS**:
- Precursor peptide sequence identification (BLAST, machine learning tools)
- PTM enzyme detection using tools: RODEO, RiPPER, RRE-Finder
- BGC boundary prediction and classification
- RiPP class assignment
- Correlation of genomic finding with bioactivity

**OUTPUT**: 
- Computationally identified RiPP BGCs correlated with bioactivity
- PTM enzyme characterization
- Precursor peptide sequences
- **Feeds into**: Silent cluster activation strategy

---

### STEP 3: Silent RiPP Activation Design

**INPUT**: 
- Characterized RiPP BGCs from Step 2
- Known RiPPs and their expression strategies

**PROCESS**:
- Computational design of BGC expression constructs
- Regulatory element engineering strategy
- Heterologous host selection and optimization
- Activation signal design (if needed)
- Expression system optimization

**OUTPUT**: 
- Expression system specifications for RiPP production
- Optimized construct designs
- Host strain specifications
- **Feeds into**: Biosynthetic engineering

---

### STEP 4: Biosynthetic Engineering and PTM Optimization

**INPUT**: 
- RiPP expression systems from Step 3
- PTM pathway information

**PROCESS**:
- Precursor peptide engineering for improved PTM efficiency
- Enzyme expression level optimization
- PTM pathway engineering
- Post-modification validation
- Yield optimization

**OUTPUT**: 
- Engineered RiPP production system with optimized yield
- Characterized RiPP products
- Production optimization parameters

---

## Final Experimental Product

**Engineered RiPP production** with:
- Novel RiPPs successfully produced from silent clusters
- Characterized bioactivity
- Scalable production systems
- Ready for therapeutic development

## Key Computational Tools

- BGC detection: antiSMASH, RiPPER, RODEO, RRE-Finder
- Precursor prediction: BLAST, HMMsearch, Machine learning tools
- Genome mining: MINER, antiSMASH modules
- Expression design: RBS Calculator, Promoter predictors
- Construct design: SnapGene, Benchling
- Sequence analysis: Biopython, seqkit
- Mass spectrometry: MZmine, XCMS
