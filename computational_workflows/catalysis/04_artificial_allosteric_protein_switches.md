# Workflow 4: Artificial Allosteric Protein Switches

**Paper**: "Artificial allosteric protein switches with machine-learning-designed receptors"

## Research Objective

- Engineer programmable allosteric protein systems using ML-designed minimal ligand-binding domains
- Create biosensors with diverse input-output modalities
- Demonstrate allostery without global conformational change through conformational entropy

## Computational Workflow

### STEP 1: ML-Designed Receptor Domain

**INPUT**: 
- Target ligand molecule (small molecule, peptide, or protein)
- Desired binding affinity specifications
- Sequence/structure constraints

**PROCESS**:
- Generative machine learning model training on binders
- Design minimal ligand-binding domain scaffolds
- Predict specificity and binding affinity
- Ensure compatibility with allosteric coupling
- Validate no global conformational change required

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
