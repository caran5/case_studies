# Workflow 1: Discovery and Engineering of RiPP Natural Products

**Paper**: "Discovery and engineering of ribosomally synthesized and post-translationally modified peptide (RiPP) natural products"

## Research Objective

- Comprehensive review of RiPP discovery using bioactivity-guided screening, genome mining, and biosynthetic engineering
- Develop strategies for activating silent RiPP clusters
- Enable systematic access to diverse RiPP natural products

## Computational Workflow

### STEP 1: RiPP Bioactivity-Guided Screening

**INPUT**: 
- Microbial genomic sequences + bioactivity screening data
- Microbial sample collection data from diverse environments

**PROCESS**:
- High-throughput screening data analysis (HiTES - high-throughput elicitor screening)
- LAESI-IMS (laser ablation electrospray ionization mass spectrometry) data processing
- MS-based dereplication against known compounds
- Identification of novel bioactive molecules
- Environmental source characterization

**OUTPUT**: 
- Identified bioactive RiPP candidates from screening
- Chemical characterization and novelty assessment
- Source organism identification
- **Feeds into**: Genome mining correlation

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
