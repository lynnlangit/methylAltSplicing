
#### PreProcessing_RNAseqData.sbatch:
Maps the RNA sequencing data to the available exon annotations from GENCODE using bedops, samtools and bedtools.
- input - output folder, first bam file (reads) and second bam file (reads)
- output - list of exons with mapped reads

#### ProbabilityInclusion.R:
Calculates the maximum a posterior probability for the inclusion of the investigated exons.
- input:
	1) data from PreProcessing_RNAseqData.sbatch
	2) cell type (IMR90, Gm12878 or H1hesc) 
- output - A table located in the directory at the end of the script containing the maximum a posterior probabilities for the inclusion of the exons.

#### PreProcessingWGBS.R:
Declares the given CpGs as mCpGs, when the methylation rate: x => t1 && x <= t2. The script can also be modified to use a smoothed distribution with the bsseq package. 
- input:
  	1) bed files (WGBS of the cell)
	2) cell type (IMR90, Gm12878 or H1hesc)
	3) lower threshold for a CpG to be defined as a mCpG (t1)
	4) upper threshold for a CpG to be defined as a mCpG (t2)
- output - A list of identified mCpGs for each cell type.

#### runCufflinks.sbatch:
Applies Cufflinks to the transcript data of the available reads.
- input - output folder, bam file, cell type (IMR90, Gm12878 or H1hesc)
- output -  Cufflinks standard output

#### AnalysisExonsIntrons.R:
Examines some characteristics of the exons and introns (like the length comparison or the Cpg/mCpG ratio comparison) and filters out some of the exons. 
- input:
	1) data from ProabilityInclusion.R
	2) data from PreProcessingWGBS.R.R
	3) data of runCufflinks.sbatch
	4) cell type (IMR90, Gm12878 or H1hesc)
- output - various plots

#### CreateFeatures.R:
Defines the feature matrix for the ANN and GBM. The cell type has to be changed in the beginning of the script.
- input - data from AnalysisExonsIntrons.R
- output - list of features for the cell type

#### WriteANNMatrix.R:
Writes the input matrix for the deep learning algorithm. Cell type can be changed in the beginning of the script.
- input - data from CreateFeatures.R
- output - Matrix for the training of an ANN.

#### AnalyseProfilesPlots:
Plots the methylation profiles before the data was fed to the ANN.
- input - data from CreateFeatures.R
- output - various plots

#### GBM_Metropolis-Hastings:
Runs a Metropolis Hastings algorithm to fine tune the parameter set of a GBM. 
NOTES: The cell type can be changed in the beginning of the script. There is also a vector (in the beginning) which selects the features for the optimisation.
- input - data from CreateFeatures.R, TRUE or FALSE parameter
			- TRUE = a default parameter set for the fine tuning found by grid search.
			- FALSE = a random parameter set and tries to fine tune the set.
- output - A log file that contains the last line of MH algorithm (best model), 
           (optional) If script is run as a batch job, then log file should contain all steps of the MH.

#### ANN_Metropolis-Hastings.R:
T script runs a Metroplis Hastings algorithm to fine tune the parameter setting for an ANN. The architecture stays constant but all other parameters changes. The cell type can be changed in the beginning of the script. The features selected for the optimisation can be changed in the section "Load Data".
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

#### ANN.R & GBM.R:
Analyses the prediction of the best ANN and the best GBM.
- input - data from WriteANNMatrix.R
- output - various plots


