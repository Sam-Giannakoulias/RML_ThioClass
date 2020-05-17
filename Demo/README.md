# Demo: How to Generate the Complex for Classification

## Preperation of the System
The following demo assumes that the following operations are done in the conda environment gila.
See __Anaconda folder__ for instruction of installing Anaconda and recieving appropriate conda environment.

## Structure Preperation
Rosetta comes with set of tools to prepare structures that we recommend. To access these tools, get a license for rosetta and pyrosetta. They can be found at tools/protein_tools/scripts/. 
### Clean PDB structures
Non-proteanacious atoms must be removed before Rosetta Simulations. 
```
python clean_pdb.py <pdb> <chain id>
```
### Renumber and Rechain PDB structures
Write a simple python script to edit the pdb files such that residues start from 1 and rechain all of the protease atoms to chain A.

### Docking
If the protease you are interested in has a structural complex with a peptide deposited in the PDB, simply trim and mutate the sequence of the experimental peptide to your sequence of interest. If an experimental complex does not exist, use PyRosetta to pose_from_sequence the peptide you are interested in. You can then relax this peptide. Manually place the peptide into the active site of the cleaned protease. At this point, for either method, FlexPepDock can be used to optimze the binding interaction.

```
'/Rosetta/main/source/bin/FlexPepDocking.static.linuxgccrelease -in:file:s ManualDockedComplex.pdb -pre_pack -pep_refine -nstruct 100 -ex1 -ex2aro -out:pdb'
```

Select the lowest enerfy complex from FlexPepDock to be relaxed with constraints.

### Relaxing the Structure
The structure must be relaxed with the following options. __Note there is no gurantee that ThioClass will be efficacious if these exact relax protocol options are not used__. The simple relax protocol is performed with constraints, beta16_cart score function, minimizer option of lbfgs_armijo_nonmonotone, minimize bond angles off and dualspace off, and the following fold tree. 

```
foldtree_array = [(active_site_position, 1, -1), 
(active_site_position, protease_end_position, -1), 
(active_site_position, amide_carbon_position, 1),
(amide_carbon_position, ligand_end_position, -1),
(amide_carbon_position, ligand_start_position, -1)]

Serine Proteases:
AtomPair  OG ActiveSiteNum C CleavedSiteNum FLAT_HARMONIC 2.75 1.0 1.0	

Cysteine Proteases:
AtomPair  SG ActiveSiteNum C CleavedSiteNum FLAT_HARMONIC 3.0 1.0 1.0
```

## Running SRS2020 ddG Predictor
There are two ways to use SRS2020 ddG predictors. The first way is running through the terminal. The second way is running through the Jupyter App interface. This is more acessible for user with limited terminal experience or those wanting a GUI interface.

Files to run this demo can be downloaded from this link due to github size limits.
https://drive.google.com/open?id=1l8n8QAy_Px3pTJ_udO0_I36LMfKv7iqd
There are two zip files that will be applicable depending on your circumstances.
1. Using Jupyter GUI App or do not have rosetta database use __SRS2020_Executable_with_database.zip__
2. Using terminal executable and have rosetta database use __SRS2020_Executable_wo_database.zip__
### Model Options
Within the models folder, there are two models with their two respective scalers. By default, the model utilized is the one trained with the training regression utilized in the published work. This can be used for other works to benchmark against SRS2020. However, we also provide that the model that is trained on the full dataset. This model should be used for production runs to predict ddG. That option can be utilized with -m and -s option specifiying the realtive path to the model and scaler __(i.e -m models/Full_Dataset_Relaxed_Beta16_Min_Model.sav -s   Full_Dataset_Relaxed_Beta16_Min_Model_Scaler.sav)__.

### Method I: The Executable
#### Accessing executable
It is important that the naming scheme for the resfile is followed. The resfile name must start with the pdb name then followed by underscore with original amino acid, its position, and the mutated amino acid. 

For example, the mutation of residue E 287 on pdb 1A4Y to A 287. The resfile will be named __1A4Y_E287A.resfile__. If one wishes to do multiple mutation, then string the mutations with underscore seperating each other. The resfile will be named __1A4Y_K500G_Y434A_Y437A.resfile__. 
All commands will be run in the terminal.
For the first time setup, one should give permissions to execute the file. On linux, this can easily be done by right clicking the executable and selecting properties and modifying permissions.
```
.\SRS2020_ddG_Predictor -r PDB_OriginalaminoacidResiduenumberMutatationaminoacid.resfile -p pdb_relaxed.pdb -db PATHTOROSETTADATABASE
```
For the specific files in this demo, the command is
```
.\SRS2020_ddG_Predictor -r 1A4Y_E287A.resfile -p 1A4Y_relaxed.pdb -db /home/sumants/anaconda3/envs/gila/lib/python3.7/site-packages/pyrosetta/database/ 
```
##### Executable Options
1. -r     Resfile       (required)
2. -p     Pdb           (required)
3. -db    Database Path (required)
4. -m     Model         (optional)
5. -scal  Scaler        (optional)

### Method II: The Jupyter App
For the first time setup, one should give permissions to execute the file. On linux, this can easily be done by right clicking the executable and selecting properties and modifying permissions. This needs to be done for SRS2020_ddG_Predictor and run_SRS2020_ddG_Predictor_App.

#### Accessing Jupyter App
The aim of this method is to allow a more accesible interface for those with limited terminal experience.

As for method I, it is important that the naming scheme for the resfile is followed. The resfile name must start with the pdb name then followed by underscore with original amino acid, its position, and the mutated amino acid. 

For example, the mutation of residue E 287 on pdb 1A4Y to A 287. The resfile will be named __1A4Y_E287A.resfile__. If one wishes to do multiple mutation, then string the mutations with underscore seperating each other. The resfile will be named __1A4Y_K500G_Y434A_Y437A.resfile__.

#### Running the Jupyter App
```
conda activate gila
.\run_SRS2020_ddG_Predictor_App
```
#### Interface
1. Click on AppMode to generate GUI interface.
![alt text](images/StartUpArrow.png "Startup")
2. Follow the instruction given in the GUI. 
![alt text](images/app_gui_int.png "GUI Interface")
3. The process will run and finish with the indication: __Done__.
