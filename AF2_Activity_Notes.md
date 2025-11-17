# Activity 1: AlphaFold2 Parameter Exploration - Instructor Notes

## Learning Objectives
1. **Understand AF2 parameters and their impacts**
2. **Develop systematic experimental design skills**
3. **Learn to critically evaluate computational predictions**
4. **Practice comparing quantitative metrics**
5. **Gain intuition for model limitations and strengths**

---

## Time Estimates
- **Setup + Part 1 (Baseline):** 30-45 min
- **Part 2 (Model comparison):** 30 min (mostly analysis, predictions already done in Part 1)
- **Part 3 (MSA impact):** 60-90 min (multiple predictions + analysis)
- **Part 4 (Templates):** 30-45 min
- **Part 5 (Advanced):** 60+ min (optional, choose subset)
- **Parts 6-8 (Comparison & synthesis):** 45-60 min

**Total:** 3-4 hours for core activity, extensible with optional challenges

---

## Computational Requirements & Setup

### IMPORTANT: Cluster-Specific Adaptations
**The activity instructions are generic and MUST be adapted to your specific cluster/compute environment.**

Each institution's computing infrastructure is different. Work with your system administrators to determine:

#### Job Submission Method
- **Recommended: `srun`** - For interactive, real-time execution
  - Allows students to see output as it runs
  - Easier debugging
  - Good for learning and troubleshooting

- **Alternative: `sbatch`** - For batch job submission
  - Use if `srun` is not available or if queue times are long
  - Requires students to check job status and outputs later
  - May need to provide job script templates

- **Example adaptations:**
  ```bash
  # Generic instruction:
  colabfold_batch input/1QYS.fasta 1_baseline/

  # With srun (interactive):
  srun --mem=16G --time=2:00:00 --cpus-per-task=8 colabfold_batch input/1QYS.fasta 1_baseline/

  # With sbatch (batch job):
  # Students would create a job script and submit with:
  sbatch run_colabfold.sh
  ```

#### GPU vs CPU
- **GPU is MUCH faster** (10-50× speedup)
  - If possible, reserve GPU nodes for this activity
  - Students will have a much better experience
  - Typical GPU job: 5-10 minutes per prediction

- **CPU is slower but functional**
  - If GPU reservation is not feasible, CPU will work
  - Typical CPU job: 30-60 minutes per prediction
  - Consider having students run fewer experiments (e.g., skip Part 5)
  - OR pre-compute some results for comparison

- **Recommendation:**
  - Ask students to run at least Part 1-2 on their own
  - Can share/pool results from Parts 3-5 to save compute time

#### Resource Allocation
Typical requirements per prediction:
- **Memory:** 8-16 GB (CPU), 16-24 GB (GPU)
- **CPUs:** 4-8 cores
- **GPU:** 1 GPU (any modern NVIDIA GPU works)
- **Time:** 5-60 min depending on GPU/CPU
- **Disk:** ~500 MB - 2 GB per prediction

#### Software Environment
- Ensure ColabFold is installed and accessible
- Verify students can load necessary modules
- Test that `colabfold_batch --help` works
- Check internet connectivity (needed for MSA generation via MMseqs2)

---

## Suggested Variations

### Time-Limited Version
**Focus on Parts 1-3 only** (model weights + MSA impact)
- Skip templates (Part 4)
- Skip advanced parameters (Part 5)
- Total time: ~2-2.5 hours

### Advanced Version
- Include all of Part 5 (recycles, relaxation, seeds)
- Add challenge extensions (Part 7)
- Have students write formal reports (Part 8)
- Total time: 5-6 hours

### Team-Based Approach
**Divide parameter space among groups:**
- Group 1: Model weights (Part 2)
- Group 2: MSA impact (Part 3)
- Group 3: Templates (Part 4)
- Group 4: Advanced parameters (Part 5)

Then pool results for class discussion (Part 6)
- Reduces individual compute time
- Encourages collaboration
- Gets broader coverage of parameter space

### Resource-Constrained Version
If compute resources are very limited:
- **Pre-compute some predictions** and provide output files
- Students analyze pre-computed results
- Focus on analysis and interpretation skills
- Have 1-2 groups run new predictions while others analyze existing data

---

## Alternative Test Proteins

### Primary recommendation: 1QYS
- Small (~100 residues)
- Single domain
- Good MSA availability
- Well-studied structure
- Fast to predict

### Alternatives if needed:
- **1UBQ** - Ubiquitin, extremely well-studied
- **2MRU** - Small, more challenging fold
- **6CRO** - DNA-binding protein, fewer homologs (tests MSA dependence)
- **8AJY** - Multimer for advanced exercises

### Considerations for protein choice:
- Size: <150 residues recommended (faster predictions)
- MSA availability: check if good homologs exist
- Structural complexity: not too simple, not too complex
- Known structure: needed for RMSD comparison

---

## Common Issues & Troubleshooting

### Installation Problems
- **Issue:** `colabfold_batch` command not found
  - **Solution:** Check module loading, PATH, or conda environment activation

- **Issue:** Missing dependencies
  - **Solution:** Ensure all ColabFold dependencies are installed (see LocalColabFold docs)

### Runtime Issues
- **Issue:** Jobs timing out
  - **Solution:** Request more time, use GPU, or reduce `--num-models`

- **Issue:** Out of memory errors
  - **Solution:** Request more memory or use CPU mode

- **Issue:** No internet access for MSA
  - **Solution:** Pre-compute MSAs or use `--max-msa 1:1` (no MSA mode)

### Output Problems
- **Issue:** Missing output files
  - **Solution:** Check that `--save-all` or appropriate flags are set

- **Issue:** Can't open PDB files
  - **Solution:** Ensure PyMOL, ChimeraX, or other visualization software is available

---

## Assessment Ideas

### Low Stakes (Formative)
- **In-class discussion participation** (Part 6)
- **Data sharing table** completion
- **Quick write-up** of key findings

### Medium Stakes
- **Lab report** with all analyses from Parts 1-4
- **Parameter comparison poster**
- **Presentation** of optimal prediction strategy

### High Stakes (Summative)
- **Full formal report** (Part 8 - Individual Report Components)
- **Prediction protocol document** for future users
- **Peer review** of another group's prediction strategies
- **Apply to novel protein** and justify parameter choices

### Optional Feedback
**Individual Report Components (Part 8) are OPTIONAL**
- Students can submit for feedback if helpful to their learning
- Not required for activity completion
- Good practice for those interested in deeper analysis

---

## Learning Outcomes Assessment

Students should be able to:
1. ✓ Execute ColabFold predictions with various parameters
2. ✓ Interpret pLDDT, PAE, and other confidence metrics
3. ✓ Compare predictions quantitatively (RMSD, scores)
4. ✓ Explain how MSAs, templates, and model weights affect predictions
5. ✓ Design prediction strategies for different use cases
6. ✓ Critically evaluate model reliability and limitations

---

## Pre-Activity Preparation

### For Instructors:
1. **Test the workflow** on your cluster with actual resource requests
2. **Prepare job script templates** if using sbatch
3. **Download test protein** (1QYS) and native structure from PDB
4. **Create example outputs** to show students what to expect
5. **Coordinate with sysadmins** for resource allocation
6. **Set up shared directory** for class data compilation (Part 6)

### For Students (assign as pre-work):
1. **Install/test visualization software** (PyMOL, ChimeraX)
2. **Review protein structure basics** (if needed)
3. **Read about AlphaFold2** (optional: original Nature papers)
4. **Familiarize with cluster** (login, file transfer, basic SLURM commands)

---

## Extension Activities

### For Fast Finishers:
- Try multimer prediction (Part 7.4)
- Test on their own project protein
- Write helper scripts to automate analysis
- Create visualization workflows

### For Deeper Learning:
- Read AlphaFold2 papers and connect to observations
- Investigate model architecture differences
- Analyze MSA quality metrics in detail
- Compare ColabFold to native AlphaFold2

### For Class Project Integration:
- Apply learned strategies to project proteins
- Build prediction pipeline for their research
- Document best practices for their lab

---

## Additional Resources for Students

### Recommended Reading:
- ColabFold paper (Mirdita et al., 2022)
- AlphaFold2 paper (Jumper et al., 2021)
- LocalColabFold documentation

### Helpful Tools:
- PyMOL tutorials for structure visualization
- Jupyter notebooks for data analysis
- Python scripts for parsing JSON output

### Online Resources:
- RCSB PDB for structure downloads
- UniProt for sequence information
- AlphaFold Protein Structure Database for reference predictions

---

## Notes on Part 8 - Individual Reports

**These are OPTIONAL and for student benefit only.**

The Individual Report Components section provides a framework for students who want to:
- Organize their findings formally
- Get instructor feedback on their analysis
- Practice scientific writing
- Build portfolio pieces

**Implementation:**
- Make clear this is optional at the start of the activity
- Offer to review submissions and provide feedback
- Do NOT grade these unless part of formal assessment
- Use as opportunity for one-on-one learning conversations

**Alternatively:**
- Use as extra credit opportunity
- Make available for students interested in deeper dive
- Provide as template for future project reports
