# AlphaFold2 Structure Prediction: Parameter Exploration Activity (BRAINSTORM)

## Overview
In this activity, all students will use **AlphaFold2 (via LocalColabFold)** to predict the structure of the **same protein (1QYS)**. By systematically varying model parameters, students will understand how different inputs and settings affect prediction quality, speed, and confidence. The class will then compare results to identify optimal prediction strategies.

---

## ⚠️ IMPORTANT: Compute Environment Notice

**These are generic instructions that MUST be adapted to your specific cluster/computing environment.**

Every institution's computing infrastructure is different. The commands shown here assume direct command-line execution, but you will likely need to adapt them for your cluster:

### Job Submission Methods

**Recommended: Use `srun` for interactive execution**
- Allows you to see output in real-time
- Easier for learning and debugging
- Example: `srun --mem=16G --time=2:00:00 --cpus-per-task=8 colabfold_batch input.fasta output/`

**Alternative: Use `sbatch` for batch job submission**
- If `srun` is not available or queue times are very long
- You'll need to create job scripts and check outputs after jobs complete
- Ask your instructor for job script templates

**Adapt all commands in this activity to your cluster's job submission system.**

### GPU vs CPU

- **GPU is MUCH faster** (10-50× speedup)
  - If your cluster allows GPU reservations for this activity, use them!
  - Predictions take ~5-10 minutes on GPU

- **CPU works but is slower**
  - If GPU reservation is not possible, CPU is fine
  - Predictions take ~30-60 minutes on CPU
  - You may need to run fewer experiments or share results with classmates


**Throughout this activity, when you see a command like:**
```bash
colabfold_batch input/1QYS.fasta 1_baseline/
```

**You may need to run something like:**
```bash
srun --mem=16G --time=1:00:00 --gpus=1 colabfold_batch input/1QYS.fasta 1_baseline/
```

**Consult with your instructor or system documentation for the correct syntax for your cluster.**

---

## Setup

### Prerequisites
1. Install LocalColabFold following: https://github.com/YoshitakaMo/localcolabfold
2. Verify installation: `colabfold_batch --help`

### Activity Directory Structure
```bash
mkdir af2_activity
cd af2_activity
```

### Download Target Protein Sequence (1QYS)
```bash
# Create input directory
mkdir input

# Download FASTA sequence
wget https://www.rcsb.org/fasta/entry/1QYS -O input/1QYS.fasta
```

**Discussion Question:**
- What is protein 1QYS?
- What is its biological function?
- Why is it a good test case for structure prediction?

---

## Part 1: Baseline Prediction & Understanding Default Behavior

**Goal:** Run a default prediction and understand what ColabFold does by default.

### 1.1 Run Basic Prediction
```bash
mkdir 1_baseline
colabfold_batch input/1QYS.fasta 1_baseline/
```

This will run with default settings:
- Automatic MSA generation via MMseqs2
- 3 recycles
- Relaxation with Amber

### 1.2 Understand What Ran
**First, verify the default behavior:**

1. **Check the output directory** - How many prediction files do you see?
   - Count the `*_unrelaxed_rank_*.pdb` files
   - Count the `*_relaxed_rank_*.pdb` files

2. **Look at the filenames** - Do they indicate different models were used?
   - You should see files like `*_unrelaxed_rank_001_alphafold2_ptm_model_1_seed_000.pdb`
   - The filename tells you: rank, model type, **which model weights (1-5)**, and seed

3. **Confirm:** By default, ColabFold runs **all 5 AlphaFold2 trained weight sets**:
   - Same neural network architecture
   - 5 different sets of trained parameters (model_1, model_2, model_3, model_4, model_5)
   - Each produces independent predictions
   - Predictions are ranked by confidence (pLDDT)

**Discussion:**
- Why would you want to run all 5 models vs. just one?
- How does ColabFold decide the ranking?

### 1.3 Examine Output Files
Files generated:
- `*_unrelaxed_rank_*.pdb` - Raw predicted structures (ranked by confidence)
- `*_relaxed_rank_*.pdb` - Energy-minimized structures (ranked)
- `*_scores_rank_*.json` - Confidence metrics for each prediction
- `*_pae_*.png` - Predicted aligned error plots
- `*_coverage.png` - MSA coverage visualization
- Potentially 5+ structures total (one per model weight set)

### 1.4 Analysis Questions

1. **Examine all model predictions:**
   - Which model (1-5) produced the rank_001 (best) prediction?
   - Which model produced the worst prediction?
   - How do you define "best"? (pLDDT, RMSD to native, visual inspection)

2. **Confidence metrics:**
   - What are the pLDDT scores? (open `*_scores_rank_001.json`)
   - Which regions have high vs. low confidence?
   - Look at the PAE plots - what do they tell you?

3. **MSA quality:**
   - Look at `*_coverage.png` - how many sequences were found?
   - What's the MSA depth?
   - Are there gaps in coverage?

4. **Compare to native structure:**
   - Download native 1QYS from PDB
   - Align predictions to native in PyMOL/ChimeraX
   - Calculate RMSD values
   - Which regions differ most from native?

**Record your baseline results:**
| Metric | Value |
|--------|-------|
| How many models ran? | |
| Best model # (rank_001) | |
| Best pLDDT (avg) | |
| Best RMSD to native | |
| MSA depth | |
| Runtime | |

---

## Part 2: Model Weight Comparison

**Goal:** Systematically compare the 5 different AlphaFold2 trained weight sets.

### 2.1 Understanding Model Weight Differences
AlphaFold2 has 5 different trained weight sets (model_1 through model_5):
- Same neural network architecture
- Trained with different random initializations
- Trained on slightly different data splits
- Each can produce different predictions for the same input
- Running multiple models and taking the best (or ensemble) improves reliability

**Note:** ColabFold uses `--num-models 5` by default (runs all 5). You can control this!

### 2.2 How to Control Which Models Run

**Understanding the parameters:**
- `--num-models N` - How many model weight sets to use (default: 5)
- `--model-order X,Y,Z` - Which specific models to run and in what order (default: 1,2,3,4,5)

**Examples:**
- `--num-models 1 --model-order 1` → Run ONLY model_1
- `--num-models 3` → Run first 3 models (1, 2, 3)
- `--model-order 1,3,5` → Run only models 1, 3, and 5 (automatically sets num-models to 3)

### 2.3 Run Each Model Individually

Let's run each model weight set separately to compare them directly.

**Step-by-step for model_1:**

1. Create directory and run:
   ```bash
   mkdir 2_model_1
   colabfold_batch input/1QYS.fasta 2_model_1/ --num-models 1 --model-order 1
   ```

2. **What this does:**
   - `--num-models 1` tells ColabFold to use only 1 model
   - `--model-order 1` specifies which one: model_1

3. **Check the output:**
   ```bash
   ls 2_model_1/
   ```

   You should see files like:
   - `1QYS_unrelaxed_rank_001_alphafold2_ptm_model_1_seed_000.pdb`
   - `1QYS_relaxed_rank_001_alphafold2_ptm_model_1_seed_000.pdb`
   - `1QYS_scores_rank_001_alphafold2_ptm_model_1_seed_000.json`
   - etc.

4. **Note the filename:** It confirms you're using `model_1`

**Repeat for the other 4 models:**

```bash
# Run model_2
mkdir 2_model_2
colabfold_batch input/1QYS.fasta 2_model_2/ --num-models 1 --model-order 2

# Run model_3
mkdir 2_model_3
colabfold_batch input/1QYS.fasta 2_model_3/ --num-models 1 --model-order 3

# Run model_4
mkdir 2_model_4
colabfold_batch input/1QYS.fasta 2_model_4/ --num-models 1 --model-order 4

# Run model_5
mkdir 2_model_5
colabfold_batch input/1QYS.fasta 2_model_5/ --num-models 1 --model-order 5
```

**After all 5 have finished running, you should have:**
- 5 directories (`2_model_1` through `2_model_5`)
- Each containing predictions from only that specific model weight set
- Now you can compare them systematically!

### 2.3 Individual Model Analysis
For each model (1-5), examine:

1. **Structure quality:**
   - RMSD to native
   - pLDDT scores
   - Visual inspection

2. **Prediction consistency:**
   - Do all 5 models agree on the overall fold?
   - Where do they differ?
   - Are differences in high or low confidence regions?

3. **Ranking comparison:**
   - Does your individual run match the ranking from Part 1?
   - Which model would rank #1 based on your RMSD measurements?

### 2.4 Comparative Questions
- Which single model would you trust most for 1QYS?
- How different are the predictions from different weight sets?
- Is there value in running all 5 models vs. just the "best" one?
- What do disagreements between models tell you?
- How much computational time does running all 5 add vs. just one?

**Create a comparison table:**
| Model | pLDDT (avg) | RMSD | Runtime | Notes |
|-------|-------------|------|---------|-------|
| model_1 | | | | |
| model_2 | | | | |
| model_3 | | | | |
| model_4 | | | | |
| model_5 | | | | |

### 2.5 Speed Test
Compare running strategies:

```bash
# Run just first 3 models (faster)
mkdir 2_three_models
colabfold_batch input/1QYS.fasta 2_three_models/ --num-models 3

# Run specific subset
mkdir 2_custom_subset
colabfold_batch input/1QYS.fasta 2_custom_subset/ --model-order 1,3,5
```

**Questions:**
- Is the quality loss worth the speed gain when using fewer models?
- What's your recommended strategy for quick screening vs. final predictions?

---

## Part 3: MSA Impact Analysis

**Goal:** Understand how Multiple Sequence Alignments affect predictions.

### 3.1 Prediction WITHOUT MSA
Run AlphaFold2 without generating MSA:

```bash
mkdir 2_no_msa

# Disable MSA search - only use single sequence
colabfold_batch input/1QYS.fasta 2_no_msa/ \
  --max-msa 1:1
```

**Note:** `--max-msa 1:1` limits to single sequence (no homologs)

### 3.2 Prediction with LIMITED MSA
Test different MSA depths:

```bash
# Small MSA (16 sequences)
mkdir 3_small_msa
colabfold_batch input/1QYS.fasta 3_small_msa/ \
  --max-msa 16:32

# Medium MSA (64 sequences)
mkdir 4_medium_msa
colabfold_batch input/1QYS.fasta 4_medium_msa/ \
  --max-msa 64:128

# Full MSA (default - typically 512)
mkdir 5_full_msa
colabfold_batch input/1QYS.fasta 5_full_msa/ \
  --max-msa 512:1024
```

### 3.3 MSA Analysis Questions

**Compare predictions across MSA depths:**

1. **Structure Quality:**
   - How does RMSD change with MSA size?
   - At what MSA depth do predictions plateau?
   - Can you get good predictions with no MSA for 1QYS?

2. **Confidence Scores:**
   - How does average pLDDT change with MSA depth?
   - Which regions benefit most from MSA information?
   - Look at PAE plots - how do they change?

3. **Computational Cost:**
   - How does runtime scale with MSA size?
   - What's the speed vs. accuracy trade-off?

4. **MSA Coverage:**
   - Look at coverage plots - which residues have good alignment depth?
   - Do gaps in MSA correspond to low confidence regions?

**Create MSA comparison table:**
| MSA Setting | Depth | pLDDT | RMSD | Runtime | Notes |
|-------------|-------|-------|------|---------|-------|
| No MSA (1:1) | | | | | |
| Small (16:32) | | | | | |
| Medium (64:128) | | | | | |
| Full (512:1024) | | | | | |

---

## Part 4: Template Usage

**Goal:** Assess impact of using structural templates from PDB.

### 4.1 Prediction WITH Templates
```bash
mkdir 6_with_templates
colabfold_batch input/1QYS.fasta 6_with_templates/ \
  --templates
```

This searches PDB for similar structures to use as templates.

### 4.2 Prediction WITHOUT Templates
```bash
mkdir 7_no_templates
colabfold_batch input/1QYS.fasta 7_no_templates/ \
  --use-templates 0
```

### 4.3 Template Analysis

**Compare with vs. without templates:**

1. **Which templates were used?**
   - Check output logs for template hits
   - What's the sequence identity of templates?
   - How many templates were found?

2. **Impact on predictions:**
   - Does using templates improve RMSD?
   - Does it improve confidence scores?
   - Do predictions change significantly?

3. **Bias considerations:**
   - Could templates introduce bias?
   - When would you want to avoid templates?
   - For 1QYS, is using templates "cheating"? (Since similar structures exist in PDB)

**Template comparison:**
| Setting | Templates Found | pLDDT | RMSD | Notes |
|---------|----------------|-------|------|-------|
| With templates | | | | |
| Without templates | | | | |

---

## Part 5: Advanced Parameter Tuning

**Goal:** Explore additional parameters that affect prediction quality and speed.

### 5.1 Number of Recycles
Recycles = how many times the model iteratively refines its prediction.

```bash
# Few recycles (fast, potentially less accurate)
mkdir 8_recycle_1
colabfold_batch input/1QYS.fasta 8_recycle_1/ \
  --num-recycle 1

# More recycles (slower, potentially more accurate)
mkdir 9_recycle_10
colabfold_batch input/1QYS.fasta 9_recycle_10/ \
  --num-recycle 10

# Default is 3 recycles
```

**Questions:**
- At what point do additional recycles stop improving predictions?
- How much does each recycle add to runtime?
- Look at intermediate structures (if saved) - how much does the structure change per recycle?

### 5.2 Amber Relaxation
Energy minimization with Amber force field.

```bash
# Without relaxation
mkdir 10_no_relax
colabfold_batch input/1QYS.fasta 10_no_relax/ \
  --amber false

# Compare to baseline (with relaxation)
```

**Questions:**
- How much does relaxation change structures?
- Does it improve geometry/clash scores?
- Is the runtime cost worth it?

### 5.3 Random Seeds
Test prediction stochasticity:

```bash
mkdir 11_seed_test

# Run same prediction with different seeds
for seed in 42 123 456 789 999; do
  colabfold_batch input/1QYS.fasta 11_seed_test/seed_${seed}/ \
    --random-seed $seed
done
```

**Questions:**
- How much do predictions vary with different random seeds?
- Are variations in high or low confidence regions?
- Should you run multiple seeds for important predictions?

---

## Part 6: Class-Wide Comparison & Discussion

### 6.1 Data Compilation
Each student/group shares their best prediction from each experiment:

| Student | Experiment | Parameters | pLDDT | RMSD | Runtime |
|---------|------------|------------|-------|------|---------|
| Group 1 | Baseline | Default | | | |
| Group 1 | No MSA | --max-msa 1:1 | | | |
| Group 1 | With templates | --templates | | | |
| ... | ... | ... | ... | ... | ... |

### 6.2 Discussion Questions

**Model Weights:**
- Did everyone get the same "best model"?
- When would you use ensemble vs. single model?

**MSA Impact:**
- What's the minimum MSA needed for 1QYS?
- How would this change for different proteins?
- What if no homologs exist? (e.g., designed proteins)

**Templates:**
- When should you use templates vs. avoid them?
- How do you know if templates are reliable?

**Computational Trade-offs:**
- What settings give best accuracy per compute time?
- Which parameters matter most for 1QYS?
- Would this generalize to other proteins?

**Prediction Confidence:**
- Can we trust pLDDT scores?
- When do models disagree? Why?
- How would you evaluate predictions without a known structure?

### 6.3 Optimal Strategy
Based on class results, define:
1. **Fast prediction strategy** (minimize time, acceptable accuracy)
2. **Accurate prediction strategy** (maximize quality, time not critical)
3. **Balanced strategy** (good accuracy, reasonable time)

---

## Part 7: Challenge Extensions (Optional)

### 7.1 Apply to Unknown Structure
Use your optimal parameters on a protein without known structure:
- Your project target protein
- A designed sequence
- A mutant of 1QYS

**Questions:**
- How do confidence scores compare to 1QYS?
- Do different parameters matter more/less?
- How do you evaluate quality without ground truth?

### 7.2 Mutation Analysis
Introduce mutations in 1QYS:
1. Choose 2-3 core residues
2. Mutate to chemically different amino acids (e.g., hydrophobic → charged)
3. Predict mutant structures with optimal parameters

**Questions:**
- Does AF2 predict destabilization?
- How much does structure change?
- Do confidence scores decrease?
- What are AF2's known limitations for mutations?

### 7.3 Minimal MSA Challenge
For 1QYS, find the absolute minimum MSA depth needed:
- Binary search: test MSA sizes until quality drops
- Plot quality vs. MSA depth
- Define threshold (e.g., RMSD < 2Å or pLDDT > 80)

**Goal:** Understand minimal information requirements.

### 7.4 Multimer Prediction
Switch to a protein complex (e.g., 8AJY):
- How do parameters affect multimer predictions differently?
- Are interface confidence scores reliable?
- Does MSA matter more for multimers?

---

## Part 8: Final Synthesis

### Class Discussion Topics

**Generalization:**
- Do 1QYS results apply to all proteins?
- How would results differ for:
  - Membrane proteins
  - Intrinsically disordered proteins
  - Proteins with few homologs
  - Large multimers

**Real-World Applications:**
- Drug discovery: which settings to use?
- Protein engineering: MSA vs. no MSA for designed proteins?
- Structural genomics: throughput vs. accuracy?

**Future Directions:**
- How could these experiments be automated?
- What additional analyses would be valuable?
- How to incorporate AF2 into research workflows?

---

## Part 9: Individual Report (OPTIONAL)

**This section is completely OPTIONAL.** If writing a formal report would be helpful to your learning, we will review it and provide feedback. Only complete this if it benefits you personally.

### Suggested Report Components

**1. Methods:**
- All experiments run
- Parameters tested
- Analysis approach

**2. Results:**
- Tables of all metrics
- Structural alignments (figures)
- Coverage/PAE plots for key comparisons

**3. Parameter Sensitivity Analysis:**
- Which parameters had biggest impact on 1QYS?
- Rank parameters by importance
- Identify parameter interactions (e.g., MSA + templates)

**4. Optimal Prediction Protocol:**
- Recommended settings for proteins like 1QYS
- Settings for fast screening
- Settings for high-accuracy final predictions

**5. Insights & Limitations:**
- What did you learn about AF2?
- When is AF2 most/least reliable?
- What questions remain?

**If you choose to write a report, feel free to submit it to us and we'll provide detailed feedback to support your learning.**
