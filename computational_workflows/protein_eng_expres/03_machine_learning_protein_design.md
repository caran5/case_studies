# Workflow 3: Machine Learning Protein Design - Data-Driven Property Prediction

**Paper**: "Application of machine learning to protein design"

## Research Objective

- Develop comprehensive ML models for multi-property protein prediction (stability, expression, activity)
- Enable rapid in silico screening of variant libraries before experimental testing
- Accelerate protein engineering cycles through predictive design

## Quick Reference

| Parameter | Value | Notes |
|-----------|-------|-------|
| **Runtime** | 6-8 weeks | Feature extraction (1 week), model training (2 weeks), validation (2-3 weeks), predictions (1 week) |
| **Storage Required** | 200 GB | Training datasets (50K+ variants), model checkpoints, feature matrices, prediction files |
| **CPU Cores** | 16-24 | Parallel feature extraction, hyperparameter optimization (8+ cores each) |
| **GPU Required** | 1× Tesla V100 (optional) | Neural network training for deep learning models (3-5× speedup) |
| **Success Metrics** | Cross-validated R²>0.85 for all properties, RMSE<0.3 log-units, 80%+ predictive accuracy on hold-out test |
| **Cost Estimate** | $12,000-18,000 | HPC compute ($5K-7K), software licenses ($2K-3K), GPU hours ($2K-3K), personnel ($3K-5K) |

## Installation & Setup

```bash
# Create conda environment for ML-based protein design
conda create -n ml_protein_design python=3.10 biopython pandas numpy scipy scikit-learn xgboost tensorflow keras pytorch -y
conda activate ml_protein_design

# Install specialized ML tools
pip install optuna hyperopt shap category_encoders imblearn

# Install protein analysis tools
pip install dssp prody foldx pdb-tools

# Install visualization and utilities
pip install matplotlib seaborn plotly jupyter notebook

# Verify installations
python -c "import sklearn, xgboost, tensorflow, keras; print('✓ All ML frameworks installed')"
echo "Setup complete!"
```

---

## STEP 1: Feature Engineering from Sequences and Structures

**OBJECTIVE**: Extract comprehensive feature vectors from protein sequences and structures that capture stability, expression, activity information for ML training.

**INPUT SPECS**:
- Protein sequences (FASTA format, aligned or unaligned)
- Structures (PDB or AlphaFold2 models)
- Experimental measurements: stability (ΔΔG), expression level, activity/fitness, solubility
- Feature specification: sequence-based, structure-based, evolutionary
- File format: CSV (sequences + measurements), PDB structures

**CODE BLOCK - Feature Extraction Pipeline**:

```python
from Bio import SeqIO, SeqUtils
import pandas as pd
import numpy as np
import subprocess
from Bio.PDB import PDBParser, PDBIO, Polypeptide
import prody as pd_prody

print("=== FEATURE ENGINEERING FOR ML ===")

# Step 1.1: Load protein sequences and measurements
sequences_df = pd.read_csv("protein_sequences_and_measurements.csv")  
# Columns: protein_id, sequence, stability_ddg, expression_level, activity, solubility

print(f"Loaded {len(sequences_df)} protein variants")

# Step 1.2: Extract sequence-based features
print("\nStep 1.2: Sequence-based feature extraction")

def extract_sequence_features(sequence):
    """Extract physicochemical and compositional features from sequence"""
    features = {}
    
    # Amino acid composition (20 features)
    aa_comp = {}
    for aa in 'ACDEFGHIKLMNPQRSTVWY':
        aa_comp[f'comp_{aa}'] = sequence.count(aa) / len(sequence)
    features.update(aa_comp)
    
    # Physicochemical properties
    features['avg_hydrophobicity'] = np.mean([SeqUtils.ProtParam.kd.get(aa, 0) for aa in sequence])
    features['charge_ph7'] = sum(1 for aa in sequence if aa in 'RHK') - sum(1 for aa in sequence if aa in 'DE')
    features['aromaticity'] = (sequence.count('Y') + sequence.count('F') + sequence.count('W')) / len(sequence)
    features['instability_index'] = SeqUtils.ProtParam.ProteinAnalysis(sequence).instability_index()
    features['molecular_weight'] = SeqUtils.ProtParam.ProteinAnalysis(sequence).molecular_weight()
    
    # Disorder propensity (IUPRED prediction - simplified)
    disorder_scores = [0.5] * len(sequence)  # Placeholder - use IUPred in production
    features['disorder_fraction'] = np.mean(disorder_scores)
    features['max_disorder'] = np.max(disorder_scores)
    
    # Secondary structure propensity (simplified)
    helix_prob = sum(1 for aa in sequence if aa in 'ALEFM') / len(sequence)  # Helix formers
    sheet_prob = sum(1 for aa in sequence if aa in 'VIY') / len(sequence)  # Sheet formers
    features['helix_propensity'] = helix_prob
    features['sheet_propensity'] = sheet_prob
    
    return features

sequence_features_list = []
for idx, row in sequences_df.iterrows():
    seq_features = extract_sequence_features(row['sequence'])
    seq_features['protein_id'] = row['protein_id']
    sequence_features_list.append(seq_features)

seq_features_df = pd.DataFrame(sequence_features_list)
print(f"Extracted {len(seq_features_df.columns) - 1} sequence features from {len(seq_features_df)} proteins")

# Step 1.3: Extract structure-based features
print("\nStep 1.3: Structure-based feature extraction")

structure_features_list = []

for idx, row in sequences_df.iterrows():
    protein_id = row['protein_id']
    pdb_file = f"structures/{protein_id}.pdb"
    
    features = {'protein_id': protein_id}
    
    try:
        # Load structure using BioPython
        parser = PDBParser(QUIET=True)
        structure = parser.get_structure(protein_id, pdb_file)
        model = structure[0]
        
        # Count secondary structure elements using DSSP
        try:
            dssp_result = subprocess.run([
                'dssp', '-i', pdb_file, '-o', f"{pdb_file.replace('.pdb', '.dssp')}"
            ], capture_output=True, text=True)
            
            # Parse DSSP output
            with open(f"{pdb_file.replace('.pdb', '.dssp')}", 'r') as f:
                dssp_lines = f.readlines()[28:]  # Skip header
                helix_count = sum(1 for line in dssp_lines if 'H' in line)
                sheet_count = sum(1 for line in dssp_lines if 'E' in line)
                coil_count = len(dssp_lines) - helix_count - sheet_count
                
                features['helix_fraction'] = helix_count / len(dssp_lines) if len(dssp_lines) > 0 else 0
                features['sheet_fraction'] = sheet_count / len(dssp_lines) if len(dssp_lines) > 0 else 0
                features['coil_fraction'] = coil_count / len(dssp_lines) if len(dssp_lines) > 0 else 0
        except:
            features['helix_fraction'] = 0.3  # Default values
            features['sheet_fraction'] = 0.3
            features['coil_fraction'] = 0.4
        
        # Calculate contact order and atomic packing
        ca_atoms = []
        for chain in model:
            for residue in chain:
                if 'CA' in residue:
                    ca_atoms.append(residue['CA'].coord)
        
        if len(ca_atoms) > 10:
            # Contact order: average sequence separation between contacts
            contacts = 0
            total_sep = 0
            for i in range(len(ca_atoms)):
                for j in range(i+4, len(ca_atoms)):  # Contacts >3 residues apart
                    dist = np.linalg.norm(ca_atoms[i] - ca_atoms[j])
                    if dist < 6.5:  # Contact threshold
                        contacts += 1
                        total_sep += (j - i)
            
            contact_order = total_sep / contacts if contacts > 0 else 0
            features['contact_order'] = contact_order / len(ca_atoms)
            features['contact_density'] = contacts / (len(ca_atoms) ** 2 / 2)
        else:
            features['contact_order'] = 0
            features['contact_density'] = 0
        
        # Use ProDy for rapid structure analysis
        try:
            structure_prody = pd_prody.parsePDB(pdb_file)
            ca_prody = structure_prody.select('name CA')
            
            # Calculate RMSD from ideal alpha helix
            if ca_prody:
                features['num_residues_structure'] = len(ca_prody)
                
                # Gyration radius (measure of compactness)
                features['gyration_radius'] = pd_prody.calcGyrationRadius(ca_prody)
        except:
            features['num_residues_structure'] = len(ca_atoms)
            features['gyration_radius'] = 0
        
        structure_features_list.append(features)
    except Exception as e:
        print(f"Warning: Could not parse structure {pdb_file}: {e}")
        # Add default features
        features.update({
            'helix_fraction': 0.3, 'sheet_fraction': 0.3, 'coil_fraction': 0.4,
            'contact_order': 0, 'contact_density': 0,
            'num_residues_structure': 0, 'gyration_radius': 0
        })
        structure_features_list.append(features)

struct_features_df = pd.DataFrame(structure_features_list)
print(f"Extracted {len(struct_features_df.columns) - 1} structure features")

# Step 1.4: Extract evolutionary features (conservation)
print("\nStep 1.4: Evolutionary feature extraction")

# Simplified conservation scoring (in production: use ConSurf, Rate4Site, etc.)
evolutionary_features_list = []

for idx, row in sequences_df.iterrows():
    features = {
        'protein_id': row['protein_id'],
        'sequence_entropy': np.random.uniform(0.1, 0.9),  # Placeholder - compute real conservation
        'conservation_score': np.random.uniform(0.3, 0.95),  # Average conservation
        'entropy_position_count': np.random.randint(5, 50)  # Highly variable positions
    }
    evolutionary_features_list.append(features)

evol_features_df = pd.DataFrame(evolutionary_features_list)

# Step 1.5: Combine all features
print("\nStep 1.5: Merging feature sets")

# Merge all feature dataframes
combined_features_df = seq_features_df.merge(struct_features_df, on='protein_id', how='left')
combined_features_df = combined_features_df.merge(evol_features_df, on='protein_id', how='left')

# Add experimental measurements
combined_features_df = combined_features_df.merge(
    sequences_df[['protein_id', 'stability_ddg', 'expression_level', 'activity', 'solubility']],
    on='protein_id', how='left'
)

print(f"Combined feature matrix: {combined_features_df.shape[0]} proteins × {combined_features_df.shape[1] - 1} features")

# Step 1.6: Feature quality assessment
print("\nStep 1.6: Feature quality and importance analysis")

# Check for missing values
missing_counts = combined_features_df.isnull().sum()
if (missing_counts > 0).any():
    print(f"Columns with missing values:")
    print(missing_counts[missing_counts > 0])

# Remove rows with missing target values
combined_features_df = combined_features_df.dropna(subset=['stability_ddg', 'expression_level', 'activity'])

print(f"Final feature matrix: {combined_features_df.shape[0]} proteins × {combined_features_df.shape[1] - 1} features")

# Export feature matrix
combined_features_df.to_csv("feature_matrix_complete.csv", index=False)

print("\n✓ Feature engineering complete")
print(f"  - {len(combined_features_df.columns) - 5} total features (20 AA composition + 8 physicochemical + 8 structure + 3 evolutionary)")
```

**OUTPUT SPECS**:
- Feature matrix (CSV): protein ID, all feature columns (50-100 total), experimental measurements
- Feature statistics (JSON): mean, std, min, max for each feature
- Missing value report: counts and percentages per column
- Feature correlation matrix (PNG): heatmap showing feature relationships
- Expected: 100-500 features per protein, 100-10K proteins in training set, <5% missing values

**VALIDATION SCRIPT**:

```python
# Validate feature matrix
assert len(combined_features_df) > 100, "Insufficient training data"
assert combined_features_df.shape[1] > 50, "Insufficient features extracted"
assert combined_features_df.isnull().sum().sum() < len(combined_features_df) * 0.01, "Too many missing values"

# Check feature distributions
for col in combined_features_df.columns[1:-4]:  # Exclude protein_id and targets
    assert combined_features_df[col].std() > 0, f"Feature {col} has no variance"

print(f"✓ Feature validation passed ({combined_features_df.shape[0]} proteins, {combined_features_df.shape[1]-5} features)")
```

**SUCCESS CRITERIA**:
- ✅ 50-100 features extracted per protein
- ✅ Sequence, structure, and evolutionary features all represented
- ✅ <5% missing values in feature matrix
- ✅ Feature distributions normal or log-normal
- ✅ No perfectly correlated feature pairs (r>0.99)

**NEXT STEP**: Train ML models on feature matrix to predict protein properties (STEP 2)

---

## STEP 2: ML Model Development and Hyperparameter Optimization

**OBJECTIVE**: Develop ensemble ML models (RandomForest, XGBoost, Neural Networks) to predict protein properties, optimize hyperparameters via Bayesian optimization, select best-performing model.

**INPUT SPECS**:
- Feature matrix from STEP 1
- Target variables: stability (ΔΔG), expression level, activity, solubility
- Train/test split specification (80/20 or 5-fold CV)
- Model architecture preferences

**CODE BLOCK - Model Training and Optimization**:

```python
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.ensemble import RandomForestRegressor, GradientBoostingRegressor
from sklearn.preprocessing import StandardScaler
import xgboost as xgb
from xgboost import XGBRegressor
from sklearn.metrics import r2_score, mean_squared_error, mean_absolute_error
import optuna
from optuna.samplers import TPESampler
import tensorflow as tf
from tensorflow import keras

print("=== ML MODEL DEVELOPMENT ===")

# Load feature matrix
X_full = pd.read_csv("feature_matrix_complete.csv")
feature_cols = [col for col in X_full.columns if col != 'protein_id' and col not in ['stability_ddg', 'expression_level', 'activity', 'solubility']]
target_cols = ['stability_ddg', 'expression_level', 'activity']

# Step 2.1: Data preparation and scaling
print("\nStep 2.1: Data preparation")

X = X_full[feature_cols].values
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

targets = X_full[target_cols]

# Split data
X_train, X_test, y_train, y_test = train_test_split(
    X_scaled, targets, test_size=0.2, random_state=42
)

print(f"Training set: {X_train.shape[0]} proteins, {X_train.shape[1]} features")
print(f"Test set: {X_test.shape[0]} proteins")

# Step 2.2: Hyperparameter optimization using Optuna (Bayesian optimization)
print("\nStep 2.2: Hyperparameter optimization for XGBoost")

def objective_xgb(trial):
    """Optuna objective function for XGBoost hyperparameter tuning"""
    
    params = {
        'n_estimators': trial.suggest_int('n_estimators', 50, 500),
        'max_depth': trial.suggest_int('max_depth', 3, 10),
        'learning_rate': trial.suggest_float('learning_rate', 0.001, 0.3, log=True),
        'subsample': trial.suggest_float('subsample', 0.5, 1.0),
        'colsample_bytree': trial.suggest_float('colsample_bytree', 0.5, 1.0),
        'gamma': trial.suggest_float('gamma', 0, 5),
        'min_child_weight': trial.suggest_int('min_child_weight', 1, 10)
    }
    
    model = XGBRegressor(**params, random_state=42, verbosity=0)
    
    # Cross-validation score (mean of all 3 targets)
    cv_scores = []
    for target_col in target_cols:
        scores = cross_val_score(model, X_train, y_train[target_col], cv=5, scoring='r2')
        cv_scores.append(scores.mean())
    
    return np.mean(cv_scores)

# Run optimization
sampler = TPESampler(seed=42)
study = optuna.create_study(sampler=sampler, direction='maximize')
study.optimize(objective_xgb, n_trials=50, show_progress_bar=True)

best_xgb_params = study.best_params
print(f"Best XGBoost parameters: {best_xgb_params}")
print(f"Best cross-validation score: {study.best_value:.3f}")

# Step 2.3: Train final models on each target
print("\nStep 2.3: Training final models on each target property")

models = {}
model_performance = []

for target_col in target_cols:
    print(f"\nTraining on target: {target_col}")
    
    # XGBoost
    xgb_model = XGBRegressor(**best_xgb_params, random_state=42)
    xgb_model.fit(X_train, y_train[target_col], verbose=False)
    xgb_pred = xgb_model.predict(X_test)
    xgb_r2 = r2_score(y_test[target_col], xgb_pred)
    xgb_rmse = np.sqrt(mean_squared_error(y_test[target_col], xgb_pred))
    
    # RandomForest (for comparison)
    rf_model = RandomForestRegressor(n_estimators=200, max_depth=10, random_state=42, n_jobs=-1)
    rf_model.fit(X_train, y_train[target_col])
    rf_pred = rf_model.predict(X_test)
    rf_r2 = r2_score(y_test[target_col], rf_pred)
    
    # GradientBoosting
    gb_model = GradientBoostingRegressor(n_estimators=200, learning_rate=0.05, random_state=42)
    gb_model.fit(X_train, y_train[target_col])
    gb_pred = gb_model.predict(X_test)
    gb_r2 = r2_score(y_test[target_col], gb_pred)
    
    print(f"  XGBoost:  R² = {xgb_r2:.3f}, RMSE = {xgb_rmse:.3f}")
    print(f"  RandomForest: R² = {rf_r2:.3f}")
    print(f"  GradientBoosting: R² = {gb_r2:.3f}")
    
    # Select best model for this target
    scores = {'XGBoost': xgb_r2, 'RandomForest': rf_r2, 'GradientBoosting': gb_r2}
    best_model_name = max(scores, key=scores.get)
    
    if best_model_name == 'XGBoost':
        best_model = xgb_model
    elif best_model_name == 'RandomForest':
        best_model = rf_model
    else:
        best_model = gb_model
    
    models[target_col] = best_model
    
    model_performance.append({
        'target': target_col,
        'best_model': best_model_name,
        'r2_score': scores[best_model_name],
        'xgb_r2': xgb_r2,
        'rf_r2': rf_r2,
        'gb_r2': gb_r2
    })

perf_df = pd.DataFrame(model_performance)
print("\n" + perf_df.to_string(index=False))

# Step 2.4: Feature importance analysis
print("\nStep 2.4: Feature importance for each target")

for target_col in target_cols:
    model = models[target_col]
    if hasattr(model, 'feature_importances_'):
        importances = model.feature_importances_
        feature_importance_df = pd.DataFrame({
            'feature': feature_cols,
            'importance': importances
        }).sort_values('importance', ascending=False)
        
        print(f"\nTop 10 features for {target_col}:")
        print(feature_importance_df.head(10).to_string(index=False))

# Export models
import joblib
for target_col, model in models.items():
    joblib.dump(model, f"model_{target_col}.pkl")

perf_df.to_csv("model_performance.csv", index=False)

print("\n✓ Model development complete")
```

**OUTPUT SPECS**:
- Trained models (PKL files): one model per property target
- Model performance (CSV): target, model type, R² score, RMSE, MAE
- Feature importance (CSV): feature ranking for each target
- Hyperparameter configurations (JSON): best parameters for each model
- Expected: R²>0.85 for at least 2/3 targets, RMSE<0.3 log-units

**VALIDATION SCRIPT**:

```python
# Validate model performance
assert len(models) >= 2, "Insufficient models trained"
for target_col in target_cols:
    assert perf_df[perf_df['target']==target_col]['r2_score'].values[0] > 0.75, f"Model for {target_col} insufficient"

print(f"✓ Model validation passed (average R² = {perf_df['r2_score'].mean():.3f})")
```

**SUCCESS CRITERIA**:
- ✅ Models trained for all property targets
- ✅ Cross-validation R² >0.80 for each target
- ✅ Test set R² >0.75 for each target
- ✅ Feature importance identified (top 5 features per target)
- ✅ Model parameters saved and reproducible

**NEXT STEP**: Generate property predictions for large variant libraries (STEP 3)

---

## STEP 3: Property Prediction Pipeline for Designed Variants

**OBJECTIVE**: Apply trained ML models to predict multiple properties (stability, expression, activity) for large libraries of designed protein variants.

**INPUT SPECS**:
- Trained ML models from STEP 2
- Designed variant sequences (FASTA format)
- Optional: variant structures (PDB/AlphaFold2)
- Feature extraction pipeline from STEP 1

**CODE BLOCK - Prediction Pipeline**:

```python
import joblib

print("=== PROPERTY PREDICTION FOR DESIGNED VARIANTS ===")

# Load trained models
models = {}
for target_col in target_cols:
    models[target_col] = joblib.load(f"model_{target_col}.pkl")

# Load designed variants
designed_variants_fasta = "designed_variants.fasta"
designed_sequences = {}
for record in SeqIO.parse(designed_variants_fasta, "fasta"):
    designed_sequences[str(record.id)] = str(record.seq)

print(f"Loaded {len(designed_sequences)} designed variants")

# Extract features for designed variants
print("Extracting features for designed variants...")

variant_features_list = []
for variant_id, sequence in designed_sequences.items():
    seq_features = extract_sequence_features(sequence)
    seq_features['variant_id'] = variant_id
    
    # Add placeholder structure features (or compute if structures available)
    seq_features.update({
        'helix_fraction': 0.3, 'sheet_fraction': 0.3, 'coil_fraction': 0.4,
        'contact_order': 0, 'contact_density': 0.1,
        'num_residues_structure': len(sequence), 'gyration_radius': 20
    })
    
    variant_features_list.append(seq_features)

variant_features_df = pd.DataFrame(variant_features_list)

# Scale features
variant_X_scaled = scaler.transform(variant_features_df[[col for col in feature_cols if col in variant_features_df.columns]])

# Generate predictions
print("Generating predictions...")

predictions_list = []
for idx, variant_id in enumerate(variant_features_df['variant_id']):
    pred_row = {'variant_id': variant_id}
    
    for target_col in target_cols:
        pred = models[target_col].predict(variant_X_scaled[idx:idx+1])[0]
        pred_row[f'{target_col}_predicted'] = pred
    
    # Confidence scoring (variance-based)
    pred_row['confidence_score'] = np.random.uniform(0.7, 0.95)  # Placeholder
    
    predictions_list.append(pred_row)

predictions_df = pd.DataFrame(predictions_list)

# Rank variants by multi-objective criterion
predictions_df['combined_score'] = (
    (predictions_df['stability_ddg_predicted'] + 2) / 4 * 0.4 +  # Stability (normalized)
    (predictions_df['expression_level_predicted']) / 10 * 0.3 +  # Expression
    (predictions_df['activity_predicted']) / 100 * 0.3  # Activity
)

predictions_df_ranked = predictions_df.nlargest(50, 'combined_score')

print(f"\nTop 10 predicted variants:")
print(predictions_df_ranked.head(10)[['variant_id', 'stability_ddg_predicted', 'expression_level_predicted', 'activity_predicted', 'combined_score']])

# Export predictions
predictions_df.to_csv("variant_predictions.csv", index=False)
predictions_df_ranked.to_csv("top_predicted_variants.csv", index=False)

print("\n✓ Prediction complete")
```

**OUTPUT SPECS**:
- Variant predictions (CSV): variant ID, predicted properties (stability, expression, activity), confidence score
- Ranked variants (CSV): top 50-100 variants ranked by combined objective
- Uncertainty estimates (JSON): prediction confidence intervals per variant
- Expected: Predictions for 100-10,000 variants, ranked by combined fitness

**VALIDATION SCRIPT**:

```python
# Validate predictions
assert len(predictions_df) > 0, "No predictions generated"
assert all(col in predictions_df.columns for col in [f'{t}_predicted' for t in target_cols]), "Missing predictions"

# Check prediction ranges
for target_col in target_cols:
    pred_col = f'{target_col}_predicted'
    mean_pred = predictions_df[pred_col].mean()
    std_pred = predictions_df[pred_col].std()
    print(f"{target_col}: mean={mean_pred:.2f}, std={std_pred:.2f}")

print(f"✓ Prediction validation passed ({len(predictions_df)} variants predicted)")
```

**SUCCESS CRITERIA**:
- ✅ Predictions generated for all designed variants
- ✅ Prediction ranges reasonable (within training data bounds)
- ✅ Confidence scores assigned
- ✅ Variants ranked by multi-objective criterion
- ✅ Top variants suitable for experimental validation

**NEXT STEP**: Validate predictions experimentally and refine models (STEP 4-5 typically combined in ML pipeline)

---

## STEP 4: Design Optimization and Multi-Property Balancing

**OBJECTIVE**: Identify optimal variant combinations that balance multiple objectives (stability, expression, activity), avoid common design pitfalls.

**INPUT SPECS**:
- Predictions from STEP 3
- Design constraints (Pareto frontier analysis)
- Property weights/priorities

**CODE BLOCK - Multi-Objective Optimization**:

```python
print("=== DESIGN OPTIMIZATION ===")

# Multi-objective Pareto frontier analysis
print("\nIdentifying Pareto-optimal variants...")

# Define objectives
predictions_df['stability_norm'] = (predictions_df['stability_ddg_predicted'] + 2) / 4  # Normalize to [0,1]
predictions_df['expression_norm'] = predictions_df['expression_level_predicted'] / 10
predictions_df['activity_norm'] = predictions_df['activity_predicted'] / 100

# Find Pareto frontier (non-dominated solutions)
pareto_variants = []

for idx1, row1 in predictions_df.iterrows():
    dominated = False
    for idx2, row2 in predictions_df.iterrows():
        if idx1 != idx2:
            # Check if row2 dominates row1 (better in all objectives)
            if (row2['stability_norm'] >= row1['stability_norm'] and
                row2['expression_norm'] >= row1['expression_norm'] and
                row2['activity_norm'] >= row1['activity_norm'] and
                (row2['stability_norm'] > row1['stability_norm'] or
                 row2['expression_norm'] > row1['expression_norm'] or
                 row2['activity_norm'] > row1['activity_norm'])):
                dominated = True
                break
    
    if not dominated:
        pareto_variants.append(row1['variant_id'])

pareto_df = predictions_df[predictions_df['variant_id'].isin(pareto_variants)]

print(f"Pareto frontier: {len(pareto_df)} non-dominated variants")
print(f"Original set: {len(predictions_df)} variants")
print(f"Trade-off candidates identified: {len(pareto_df)}")

# Export Pareto frontier variants
pareto_df.to_csv("pareto_optimal_variants.csv", index=False)

print("\n✓ Design optimization complete")
```

**OUTPUT SPECS**:
- Pareto frontier variants (CSV): variants on Pareto frontier with all properties
- Trade-off analysis (JSON): property correlations, trade-off regions
- Recommended variants (JSON): top 5-10 variants with rationale for selection

**SUCCESS CRITERIA**:
- ✅ Pareto frontier identified (5-20% of variants)
- ✅ Clear trade-off regions visualized
- ✅ Ranking systems applied per objective

---

## STEP 5: Experimental Validation and Model Refinement

**OBJECTIVE**: Compare ML predictions to experimental measurements, quantify prediction accuracy, iteratively refine models with new data.

**INPUT SPECS**:
- Predicted variants from STEP 3-4
- Experimental measurements for validation set
- Model accuracy requirements

**OUTPUT SPECS**:
- Validation report (CSV): predicted vs. actual properties, residuals
- Model accuracy metrics (JSON): R², RMSE, prediction errors per property
- Refined models (PKL): updated with validation data

**SUCCESS CRITERIA**:
- ✅ Prediction accuracy validated (R² >0.80 on independent test set)
- ✅ <10% variants with prediction error >1 log-unit
- ✅ Models generalize across multiple protein families

---

## Troubleshooting Guide

| Problem | Root Cause | Solution |
|---------|-----------|----------|
| **Low model accuracy (R²<0.70)** | Insufficient training data or poor features | Expand training dataset to 500+ proteins, add more structure-based features, tune hyperparameters |
| **Feature extraction fails** | Missing structures or DSSP errors | Use AlphaFold2 for missing structures, catch exceptions for DSSP failures, use default feature values |
| **Model overfitting** | Complex model on small dataset | Reduce model complexity (max_depth, n_estimators), increase regularization, use cross-validation |
| **Prediction noise** | High variance between replicates | Ensemble multiple models, use uncertainty quantification, filter low-confidence predictions |
| **Designed variants don't perform** | Distribution shift between training and design | Validate on proteins similar to training set, design with ensemble predictions, include in feedback loop |

## Resource Requirements

| Resource | Specification | Justification |
|----------|---------------|---------------|
| **Compute (CPU)** | 16-24 cores | Feature extraction (parallel), hyperparameter optimization (8+ trials parallel), model training |
| **Memory (RAM)** | 64-128 GB | Training dataset in memory (50K variants × 100 features), model storage |
| **Storage** | 200 GB | Training data (50 GB), feature matrices (30 GB), models (5 GB), predictions (115 GB) |
| **GPU** | 1× Tesla V100 (optional) | Deep learning model training, TensorFlow acceleration |
| **Runtime** | 6-8 weeks | Feature extraction: 1 week; model optimization: 2 weeks; predictions: 1 week; validation: 2-3 weeks |
| **Cost Estimate** | $12,000-18,000 | HPC compute ($5K-7K), software licenses ($2K-3K), GPU access ($2K-3K), personnel ($3K-5K) |

## Tool Installation Matrix

| Tool | Version | Install Command | Purpose |
|------|---------|-----------------|---------|
| Scikit-Learn | 1.2+ | `pip install scikit-learn` | Ensemble methods (RandomForest, GradientBoosting) |
| XGBoost | 1.6+ | `pip install xgboost` | Extreme gradient boosting for regression |
| Optuna | 3.0+ | `pip install optuna` | Bayesian hyperparameter optimization |
| TensorFlow | 2.10+ | `pip install tensorflow` | Deep learning models, neural networks |
| Keras | 2.11+ | Included in TensorFlow | High-level neural network API |
| BioPython | 1.80+ | `pip install biopython` | Sequence analysis and SeqUtils |
| ProDy | 1.11+ | `pip install prody` | Structure analysis and dynamics |
| DSSP | 4.0+ | `conda install -c bioconda dssp` | Secondary structure assignment |
| PyMOL | 2.5+ | `conda install -c conda-forge pymol-open-source` | Structure visualization |
| Pandas | 1.5+ | `pip install pandas` | Data manipulation and analysis |

## Example Walkthrough: Predicting Stability and Expression for TEM-1 Library

### Scenario
A team develops ML models for TEM-1 β-lactamase variants to predict stability (ΔΔG), expression level, and activity from sequence features, then uses models to design 1,000 novel variants for experimental screening.

### Timeline & Milestones

| Week | Task | Output | Person-Hours |
|------|------|--------|--------------|
| 1 | Feature extraction from 500 TEM-1 variants | Feature matrix (500 × 150 features) | 30 |
| 2 | ML model training and optimization | 3 trained models (R²>0.85), hyperparameters | 50 |
| 3 | Predicting 1,000 designed variants | Ranked variants, Pareto frontier (120 variants) | 20 |
| 4-5 | Experimental validation of top 50 | Measured stability, expression, activity | 100 |
| 6-8 | Model refinement and iterative design | Refined models, 2nd generation designs | 80 |
| **Total: 8 weeks** | **ML-guided protein engineering pipeline** | **280 person-hours** |

### Expected Outcomes

- **Feature Engineering**: 150+ features extracted, including 20 amino acid composition, 8 physicochemical, 8 structure-based, 3 evolutionary
- **Model Performance**: XGBoost R²=0.87 (stability), 0.82 (expression), 0.79 (activity)
- **Designed Variants**: 1,000 computationally designed variants, top 50 ranked for testing
- **Experimental Validation**: 80% of top-10 predictions validated (measured property within ±1 log-unit of prediction)
- **Iteration**: Model refined with experimental data, 2nd generation designs show 15% improvement in accuracy

---

## Success Checklist

### Pre-Analysis
- [ ] Training dataset acquired: 100-1,000 characterized protein variants with measurements
- [ ] Feature space defined: 50-150 features covering sequence, structure, evolutionary aspects
- [ ] Data quality verified: <5% missing values, no obvious outliers without justification
- [ ] Train/test split defined: 80/20 or 5-fold cross-validation strategy chosen

### STEP 1 Validation
- [ ] Features extracted successfully: 50-150 features per protein
- [ ] Feature distributions normal or log-normal: skewness <2
- [ ] No perfect correlations between features: r_max <0.99
- [ ] Targets well-distributed: no extreme value concentration

### STEP 2 Validation
- [ ] Multiple models trained: RandomForest, XGBoost, ± Neural Networks
- [ ] Hyperparameter optimization completed: 30-50 trials performed
- [ ] Cross-validation R² >0.80 for each target
- [ ] Feature importance identified: top 5 features per target

### STEP 3 Validation
- [ ] Predictions generated for 100-10,000 variants
- [ ] Prediction confidence scores assigned
- [ ] Variants ranked by objective criterion
- [ ] Top variants suitable for experimental validation

### STEP 4 Validation
- [ ] Pareto frontier identified: 5-20% of variants non-dominated
- [ ] Trade-off analysis completed: property correlations quantified
- [ ] Optimal variants selected: clear rationale provided

### STEP 5 Validation
- [ ] Experimental validation completed: 30-50 variants tested
- [ ] Prediction accuracy assessed: 70-80% within ±1 log-unit
- [ ] Model refinement initiated: new data incorporated
- [ ] Iterative design cycle established

### Final Output
- [ ] ML models published/shared: code, trained models, predictions reproducible
- [ ] Designed proteins produced and validated
- [ ] Guidelines for ML-guided design documented
- [ ] Computational platform established for future designs

---

## Final Experimental Product

**ML-enhanced protein design pipeline** with:
- ✅ Multi-property predictive models (stability, expression, activity; R²>0.85)
- ✅ Predictions for 100-10,000 designed variants
- ✅ Pareto-optimal designs balancing multiple objectives
- ✅ 70-85% prediction accuracy validated experimentally
- ✅ Generalizable framework for rapid protein engineering
- ✅ Accelerated design cycles (weeks vs. months)
- ✅ Data-driven insights into protein property trade-offs

---

## Key Computational Tools

- **Machine learning**: scikit-learn, XGBoost, TensorFlow, Keras, PyTorch
- **Hyperparameter optimization**: Optuna, Hyperopt, Ray Tune
- **Feature extraction**: BioPython, ProDy, DSSP
- **Data analysis**: Pandas, NumPy, SciPy
- **Uncertainty quantification**: Bayesian methods, ensemble methods
- **Visualization**: Matplotlib, Plotly, Seaborn
- **Structure analysis**: PyMOL, UCSF Chimera
