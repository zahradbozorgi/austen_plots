# austen_plots
## Introduction
This repository contains demo data and code for  
[Sense and Sensitivity Analysis: Simple Post-Hoc Analysis of Bias Due to Unobserved Confounding](https://arxiv.org/abs/2003.01747)  
_Victor Veitch and Anisha Zaveri_

If a common cause affects both a treatment and outcome it can induce a spurious correlation. 
For example, the wealth of a patient influences both their health outcomes and whether they take an expensive drug.
The presence of the common cause (wealth) induces a spurious positive association between the drug and the outcome.
Austen plots are a simple visual method of determining whether some unobserved common could explain away the association between a specified treatment and outcome. The included software produces Austen plots from the outputs of standard data modeling used in causal inference pipelines. 

## Requirements
The code has been tested on Python 3.8.3 with the following Python packages:  
scikit-learn==0.23.1  
scipy==1.5.0  
numpy==1.18.5  
pandas==1.1.13  
plotnine==0.7.1 (https://pypi.org/project/plotnine/)  
tqdm==4.47.0 (https://pypi.org/project/tqdm/)  


## Instructions
### Without Bootstrapping
Use files under `example/` as reference:
1) Fit your data using any model and generate predictions for g, the propensity score, and Q, the conditional expected outcome.
2) Generate a .csv file with the following columns: 'g', 'Q', 't', 'y'. These correspond to the propensity score, the conditional expected outcome, the treatment and the outcome. For reference look at `input_df.csv` provided under `example/`.
3) (Optional, but recommended) Repeat step 1 with key covariates dropped before model fitting. For each such instance, generate a .csv file similar to step 2. Save all such files under a single folder (called `covariates` in the example). If you name one of these files 'treatment.csv', the code assumes that these are predictions generated from data without treatment, and thus this is not plotted on the graph. However, the Rsqhat value for 'treatment' is provided in the output co-ordinates file.
4) Decide a meaningful amount of bias you would like to test for, based on domain knowledge about your dataset. Let's fix this as 2 for the example dataset
5) Run the following code (values correspond to the example dataset).  

`do_sensitivity.py -input_csv input_df.csv -bias -2 -output_dir sensitivity_output -covariate_dir covariates/`

The output is an austen plot. Additionally, co-ordinates for the plot are provided as .csv files.

By default `do_sensitivity.py` calculates bias for the ATE. If you would like to calculate bias for the ATT (i.e. restrict sensitivity analysis to datapoints in your `input.csv` file where t=1) add the flag `-do_att True` to the command above.

### With Bootstrapping
You can optionally decide to generate plots with bootstrap confidence intervals. For this, after doing the steps 1-4 under the section 'Without bootstrapping' do the following:
1) Create a folder for bootstrapped inputs/outputs. In the example this is called `bootstrap`.
2) Within `bootstrap` create subfolders for each bootstrap iteration.
3) Within each boostrapped iteration folder save .csv and optionally, covariate files, as described in steps 1-3 using 'g', 'Q', 't', 'y' values obtained from a bootstrapped dataset. These should have the same names as those in the parent folder. 
_Recommendation_: If you are generating these values using cross validation techniques on a model, ensure that replicate rows generated by the bootstrapping procedure are within the same fold.
4) (Optional) Decide values for confidence interval cutoffs, and multiprocessing. Default value for the confidence interval cutoff is 0.95. Default value for multiprocessing is 1, you can set this to the [number of cores](https://www.w3resource.com/python-exercises/python-basic-exercise-47.php) on your computer to maximize speed.
5) Run the following code (values correspond to the example dataset). You may specify optional flags `-cut`, `-multi`, `-do_att` as needed.

`do_sensitivity.py -input_csv input_df.csv -bias -2 -output_dir sensitivity_output -covariate_dir covariates/ -boot bootstrap/` 

The outputs are individual austen plots for each bootstrapped dataset along with a consolidated plot showing confidence intervals. Additionally, co-ordinates for the plots are provided as .csv files. 

