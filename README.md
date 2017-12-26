
## Workflow Process

'0. PreProcessing_RNAseqData.sbatch  
1a. ProbabilityInclusion.R 1b. PreProcessingWGBS.R 1c. runCufflinks.sbatch  
'2. AnalysisExonsIntrons.R  
'3. CreateFeatures.R   
4a. WriteANNMatrix.R 4b. AnalyseProfilesPlots --- 4c. GBM_Metropolis-Hastings.R    
5a. ANN_Metropolis-Hastings.R 5b. ANN.R 5c. GBM.R

#### 0. PreProcessing_RNAseqData.sbatch: Maps RNA sequencing data to available exon annotations from GENCODE 
Note: Uses bedops, samtools and bedtools
- input - output folder, first bam file (reads) & second bam file (reads)
- output - list of exons with mapped reads

#### 1a. ProbabilityInclusion.R: Calcs max a posterior probability for inclusion of investigated exons
- input:
	1) data from PreProcessing_RNAseqData.sbatch
	2) cell type (IMR90, Gm12878 or H1hesc) 
- output - table (in directory end of script) w/ max a posterior probabilities for inclusion of exons

#### 1b. PreProcessingWGBS.R: Declares given CpGs as mCpGs, for methylation rate: x => t1 && x <= t2. 
Note: Script can also be modified to use smoothed distribution w/ bsseq package. 
- input:
  	1) bed files (WGBS of the cell)
	2) cell type (IMR90, Gm12878 or H1hesc)
	3) lower threshold for a CpG to be defined as a mCpG (t1)
	4) upper threshold for a CpG to be defined as a mCpG (t2)
- output - A list of identified mCpGs for each cell type.

#### 1c. runCufflinks.sbatch: Applies Cufflinks to transcript data of available reads
- input - output folder, bam file, cell type (IMR90, Gm12878 or H1hesc)
- output -  Cufflinks standard output

#### 2. AnalysisExonsIntrons.R: Examines some characteristics of the exons & introns 
Example: the length comparison or the Cpg/mCpG ratio comparison 
Note: filters out some of the exons. 
- input:
	1) data from ProabilityInclusion.R
	2) data from PreProcessingWGBS.R.R
	3) data of runCufflinks.sbatch
	4) cell type (IMR90, Gm12878 or H1hesc)
- output - various plots

#### 3. CreateFeatures.R: Defines feature matrix for ANN & GBM. 
Note: Cell type MUST be changed in beginning of script
- input - data from AnalysisExonsIntrons.R
- output - list of features for the cell type

#### 4a. WriteANNMatrix.R: Writes input matrix for deep learning algorithm. 
Note: Cell type CAN be changed in beginning of script.
- input - data from CreateFeatures.R
- output - Matrix for the training of an ANN.

#### 4b. AnalyseProfilesPlots: Plots methylation profiles before data is fed to the ANN.
- input - data from CreateFeatures.R
- output - various plots

#### 4c. GBM_Metropolis-Hastings.R: Runs Metropolis Hastings algorithm to fine tune param set of a GBM 
NOTES: The cell type can be changed in the beginning of the script. 
There is also a vector (in the beginning) which selects the features for the optimisation.
- input - data from CreateFeatures.R, TRUE or FALSE parameter
			- TRUE = a default parameter set for the fine tuning found by grid search.
			- FALSE = a random parameter set and tries to fine tune the set.
- output - A log file that contains the last line of MH algorithm (best model), 
           (optional) If script is run as a batch job, then log file should contain all steps of the MH.

#### 5a. ANN_Metropolis-Hastings.R: Runs Metroplis Hastings algo to fine tune param setting for an ANN
Notes: The architecture stays constant but all other parameters changes. The cell type can be changed in the beginning of the script. The features selected for the optimisation can be changed in the section "Load Data".
- input:
	1) data from WriteANNMatrix.R
  	2) units (number of units in the hidden layer)
  	3) batch size (start value of the batch size)
  	4) rate (start value of the learning rate)
  	5) dropout (start value of the dropout) 
  	6) regularisation (start value of the l2 regularisation) 
  	7) momentum (start value of the momentum parameter)
- output:
	1) A log file that contains the last line of MH algorithm (best model).
	2) (optional) If the script is run as a batch job, then the log file should contain all steps of the MH.

#### 5b & 5c. ANN.R & GBM.R: Analyses prediction of best ANN and best GBM.
- input - data from WriteANNMatrix.R
- output - various plots


