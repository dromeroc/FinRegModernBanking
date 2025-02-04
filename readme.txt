This file provides step by step instructions to replicate all results in the paper, as submitted for publication to Review of Economic Studies.




%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
        Part 1: Solve and Simulate the Model
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

This part can be done locally or on a HPCC cluster. Instructions below are for local execution. If recomputing all 40 economies (=paramater combinations) required for the paper, it is much faster to do it on the cluster.

------------------------------
Part 1a: Computing locally
------------------------------

Repeat the steps below for each economy in experdef_20201117.txt. The definitions of these economies are in experdef_20201117.m. Let the placeholder "econ" represent the name of the economy from this text file.

Run the economy for up to 300 iterations.
1.  Configure mainSB_create_env.m as follows (leave other variables as is):
	- expername = 'econ';
	- guess_mode = 'no_guess';
2.  Run mainSB_create_env.m and it will produce a file called env_econ.mat
3.  Configure main_run_exper.m as follows (leave other variables as is):
	- no_par_processes = XX; % where XX is the number of processors on the machine you're running it on
	- exper_path = 'env_econ.mat';
	- maxit = 300;
4.  Run main_run_exper.m. On a machine with 16 cores, this should take about 1 hour. It will create a file named res_TIMESTAMP.mat, where TIMESTAMP is the time at which the calcuation is finished expressed as YYYY_MM_DD_hh_mm
5.  Rename the res_TIMESTAMP.mat file to res_2020117_econ.mat.

Next, simulate. The model must be solved and res* file must exist.
6. Configure sim_stationary.m as follows (leave other variables as is):
	- resfile = 'res_2020117_econ';
7. Run sim_stationary.m. If you are running sim_stationary having run it before during the current MATLAB session, run "clear" beforehand. This operation will create the following files:
	- sim_res_20201117_econ.mat: all simulation results incl. full series, statistics, errors, and parameters
	- Results/statsexog_res_20201117_econ.xls: statistics using exogenous subsampling to define crises (one sample per worksheet)
	- Results/errstats_res_20201117_econ.xls: statistics of EE errors

Next, compute impulse response functions. Model must be solved and simulated. Both res* and sim_res* files must exist.
8. Configure sim_trans.m as follows (leave other variables as is):
	- resfile_list = {'res_20201117_econ'};	
9. Run sim_trans.m. On a machine with 16 cores, this should take about 10 min. It will create the following files:
	- GR_res_20201117_econ.mat: mean, median, and sd of IRF paths for each of 4 shocks (no shock, recession, run recession)
	
------------------------------------
Part 1b: Computing on the cluster
------------------------------------

The following instructions work for a HPCC cluster that uses a job scheduling engine for batch jobs. The code folder contains a sample batch script 'hpcc_script_aws_array.sh' used to run all 40 economies as part of a single batch job on Wharton's HPCC using dedicated AWS nodes. 

The configuration needed to run the numerical experiments as batch job on a cluster will vary substantially based on the specific cluster configuation. However, the basic structure should be transferable.  



%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
			Part 2: Compute and Save Results
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

This part pre-supposes that the res* and sim_res* for each economy created by Part 1 exist. NOTE: the .mat files for all economies are available as zip archive "res_sim_files.zip" with the supplemental materials via the Review of Economic Studies publication.

The steps below re-create all the results used in the paper.
	
1.  Run writeStats.m. This will create Results/simres.mat, which contains a variable called "mapDict," a hash-map of every simulation and IRF statistic and model parameter. They can be retrieved by running mapDict("key"), where "key" has the following format:
	[command] - [econ] - [subsmaple] - [variable] - [statistic]
	
	command:
		- sim: simulation statistic
		- comp: simulation statistic relative to bench (either level or percent change)
		
	econ: name of the economy (as in experdef_20200910.txt)
	
	subsample: if command is "sim" or "comp"
		- u: unconditional
		- e: expansions 
		- r: recessions
		- c: crises (ie run recessions)
		
	variable: all variables reported by sim_stationary (as well as welfare "VHcons" for unconditional mean) 
	
	statistic: for command = "sim" and "comp", all statistics reported by sim_stationary e.g. "mean," "std, "AC". 
			
	
2.	After simres.mat has been created in step 1, use build.m script in subfolder "paper_tables" to fill in placeholders in the file "BLrev_v2021_tables.tex". To do so, open Matlab to this folder and execute: 
	
	build('BLrev_v2021_tables','../Results/simres.mat',[])
	
	This will create a new file "BLrev_v2021_tables_filled.tex", in which placeholders are replaced by the numbers from model simulations as in the paper. This file can be compiled in a standard Latex environment.	
		
3.  Compute base economy IRF graphs, Figure 1 in the paper.
		a. Configure sim_trans.m as follows:
			- resfile_list = {'res_2020117_base'};
			- no_par_processes = XX;  % where XX is the number of processors on the machine you're running it on
		b. Run sim_trans.m. This will create GTR_res_20201117_base.mat.
		c. Configure plot_trans.m as follows:
			- resfile = 'res_20201117_base';
		   Running plot_trans.m will create Figure 1.(pdf|eps).

4.  Compute financial crisis simulations graph, Figure 2 in the paper.
		a. Configure sim_trans_GFC.m as follows:
			- resfile_start = 'res_20201117_precrisis';
			- resfile_end = 'res_20201117_postcrisis';
			- resfile_steps={'res_20201117_postcrisis085',...
               'res_20201117_postcrisis085',...
               'res_20201117_postcrisis09',...
               'res_20201117_postcrisis09',...
               'res_20201117_postcrisis095',...
               'res_20201117_postcrisis095',...
               'res_20201117_postcrisis10',...
               'res_20201117_postcrisis10',...
               'res_20201117_postcrisis105',...
               'res_20201117_postcrisis105'};
			- no_par_processes = XX;  % where XX is the number of processors on the machine you're running it on
		b. Run sim_trans_GFC.m. This will create GFC_res_20201117_postcrisis.mat.
		c. Configure sim_trans_GFC.m as follows:
			- resfile_start = 'res_20201117_precrisis';
			- resfile_end = 'res_20201117_postcrisis08';
			- resfile_steps={'res_20201117_postcrisisc1',...
               'res_20201117_postcrisisc1',...
               'res_20201117_postcrisisc2',...
               'res_20201117_postcrisisc2',...
               'res_20201117_postcrisisc3',...
               'res_20201117_postcrisisc3',...
               'res_20201117_postcrisisc4',...
               'res_20201117_postcrisisc4',...
               'res_20201117_postcrisisc5',...
               'res_20201117_postcrisisc5'};
			- no_par_processes = XX;  % where XX is the number of processors on the machine you're running it on
		d. Run sim_trans_GFC.m. This will create GFC_res_20201117_postcrisis08.mat.
		e. Run plot_trans_GFC.m. This scripts reads the contents of the two GFC_ files generated above and also "Post Crisis.xlsx" that contains the time series for the data lines in the plot. It will create Figure 2.(pdf|eps).
