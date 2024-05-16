# eDNA_workshop 
<img src="https://qubs.ca/themes/qubs/logo.png" width="270" height="100"> <img src="https://qubs.ca/themes/qubs/images/queens-logo.png" width="150" height="100"> <img src="https://carleton.ca/engineering-design/wp-content/uploads/logo-nserc-400x300.png" width="150" height="150"> 


material for [eDNA workshop 2023](https://qubs.ca/eDNAWorkshop)

# Logging in
•	Follow steps emailed to you from the Centre for Advanced Computing (CAC) to activate your account. You will need to keep note of your username and password.
•	Login to the CAC cluster via the terminal (Mac users) or mobaXterm (PC users).
```command
ssh -X username@login.cac.queensu.ca
password: (type password, it will remain hidden while you type) 
```
List directories to see what is already in your account (there should be anything).
```command
ls
````
•	We will first have to prepare our workspace by downloading some dependancies that our pipeline will run for us as well as any data that we are going to use.

# Preparing your workspace
## Install FLASH 
We will open a text editor to copy and paste the following script. To open a new text editor type `nano` on your home directory and click enter. Copy and paste the following code chunk then click `control + x` to exit, you will be asked to save changes, click `y` for yes. Save as `InstallBarque.sh`.
``` command
export INSDIR=$HOME/software
mkdir -p $INSDIR
module --force purge
module load StdEnv/2020 gcc/9.3.0
module load python/3.7
module load r/4.1.2
module load java
module load vsearch/2.21.1 flash

cd $INSDIR
git clone https://github.com/torognes/swarm.git
cd swarm/
make -j4
cd $INSDIR
git clone https://github.com/enormandeau/barque
```
We now have to make sure we have to appropriate modification access to this file. We can do this using `chmod` and make sure we can execute by adding `+x` then the file name. 
```command
chmod +x ./InstallBarque.sh
```
Now we can execute the file using the code below:
```command
./InstallBarque.sh
```
## Create Bash Script
We will use `Bash` to submit our request to run our pipeline on a different node. We could run it directly on our login node but this can slow down the node for others trying to login or manage their files. To do this we will make a ‘Bash script’ in a text editor and specify it as a script with `.sh` at the end of the file name.

•	Make a .sh file to submit as a job: use the `nano` function to open a text editor.

•	Fill parameters as needed. Below is my basic job.
```command
#!/bin/bash
#SBATCH --reservation=teaching
#SBATCH --account=teaching
#SBATCH -J barque_test #job name
#SBATCH -N 1 #min number of nodes for job
#SBATCH -n 1 #max number of tasks
#SBATCH -c 1 #CPUs per task
#SBATCH --mem= 2048 MB #memory required per node 
#SBATCH -t 0-1:0:0 #max time will take = days-hours:mins:secs
#SBATCH --mail-type=BEGIN,END,FAIL #messages you want to receive via email about job
#SBATCH --mail-user=youremail@example.ca #where to send those emails
#SBATCH -o barque_test_done.o #output filename
cd barque
module load StdEnv/2020 vsearch/2.15.2 gcc/9.3.0 r/4.0.2 flash/1.2.11
./barque 02_info/barque_config.sh
```
•	Change the email you want the confirmation reports to go to.

•	Once your bash script is customized, exit using `^x`, save with changes `y`, and name the script `run_barque.sh`. We will leave this for now as we prepare our data.

## Download Data and Database from the Github repository
Use the git clone [repository path] to download the repository containing all the data for this tutorial.
```command
git clone https://github.com/staffordmaracle/eDNA_workshop
```
Check out the data, change directories using ‘cd’ to move into the directory and list `ls` to see what you just downloaded. Exit by changing back to the previous directory ‘cd ..’.
```command
cd eDNA_workshop
ls
```
The **12S.fasta.gz** is our database in a GNU zip file. We could unzip it using `gunzip 12S.fasta.gz` to look at the species in our database. For now we will keep it zipped. If you open it `gzip 12S.fasta.gz` will rezip it.

The **Raw_reads** folder contain raw reads for each of our samples. You can look at these by changing directory `cd Raw_reads` and list `ls` to see the gzipped fasta files. `cd ..` to leave the Raw_reads directory.

You should be in the base directory `[sa125050@caclogin03]$`. If you need to leave the **eDNA_workshop** directory as well, use `cd ..` to go back out one level.

# Preparing a run
## Create a local copy of the [Barque pipeline](https://github.com/enormandeau/barque) 
•	Download a copy of the **Barque** repository using the git clone function again below. 
```command
git clone https://github.com/enormandeau/barque
```
• Look around the directory using `cd barque` before moving on. remember `ch filename` will move you into a directory and `cd ..` will move you out.

## Filling the pipeline with our data
From our base directory, copy and paste the sequences from the Raw_reads folder in the eDNA_workshop directory into the barque data folder using the `cp` command. 
*note*: `/*` means everything inside that directory.
```command
cp eDNA_workshop/Raw_reads/* barque/04_data/
```
Copy and paste the reference database from the eDNA_workshop directory into the barque database folder.
```command
cp eDNA_workshop/12S.fasta.gz barque/03_databases/
```
•	If you prepare your own database (see Formatting database below) and deposit the fasta.gz file in the `03_databases` folder and give it a name that matches the information of the `02_info/primers.csv` file.
*Ex: our database is called 12S and see below in our primer file line it says we are using the 12S database.*.

•	Edit `02_info/primers.csv` to provide information describing your primers
*Primer name, forward seq, reverse seq, Length min (=length of the target amplicon -30bp), Length ma (length of the target amplicon + 30bp), database name, thresholds species /…*.
```command
cd barque/02_info/
```
```command
nano primers.csv
  #If you have your primers saved in a backup folder just copy and paste it in the current barque run.
cp 01_barque_backup/02_info/primers.csv barque/02_info/primers.csv
`#Or copy and paste below.
12S_200pb, GTCGGTAAAACTCGTGCCAGC, CATAGTGGGGTATCTAATCCCAGTTTG,150,190,12S,0.97,0.95,0.90
```
•	The primer.csv file is accessed during the pipeline but only the lines that are not hashtag will be used. So you will want to hashtag all the primers you didn’t use and remove hashtags from the line (primers) you used.

## Specify pipeline configuration
Change the configuration file to control the specifications of each software. Nothing needs to be changed for our data but below is how you access the file to view and modify the parameters.
```command
cd barque/02_info
nano barque_config.sh
```
## Load modules (skip)
This is included in the `run_barque.sh` we made earlier so you won’t have to run the below code but note that this is happening in the background.
```command
#  module load vsearch/2.15.2
#  module load gcc/9.3.0 r/4.0.2
#  module load StdEnv/2020 gcc/9.3.0 flash/1.2.11
```
# Launch Barque
Make sure the that the configuration file name is the same in the `02_info` directory in barque and in your `run_barque.sh` file. It should just be `barque_config.sh`.

We can now ask for the resources to run the pipeline using *SLURM*, an open source scheduler. We will now use the `run_barque.sh` file we made earlier.
```command
sbatch run_barque.sh
```
The line below is built into the run_barque.sh file so you don’t need to run it but it is the one that ultimately runs the pipeline for us.
./barque 02_info/barque_config.sh

# Check your results
As barque runs through the different packages it passes the files to different folders and ultimately ending in the results directory `12_results`. If the run was completed to fruition, this directory will contain multiple files. We will check them below.

## LogFiles
The first thing you should check when your run is done is the `run.log` file. This will show you the output of each step in the workflow. You want to check that there is nothing weird that happened. This is also the place you should go if you run fails to find out why.

## Sequence Dropout
There are two files to check how many reads were removed at each step. The `sequence_dropout.csv` and `sequence_dropout_figure.png` after looking at these you may wish to alter the configuration file to relax or tighten each filtering step.

## Non annotated sequences
You will undoubtably have sequences that were not assigned to any species, phylum, or genus. These will be listed in the `most_frequently_non_annotated_sequences.fasta` file. You can BLAST the top sequences manually and check for sequences that return a single species as the likely hit (100% per idet. 100% coverage). If we get a BLAST result for non-annotsted sequence, that species is not in your databases so it was unable to assign it. We can copy the species sequence and navigate to your database file and input the sequence then rerun the analysis.

## Species Table ###(phylum and genus if you want higher classification)
This is the OTU table you will use for further analysis. You can take a quick look at the species table by using `nano`. This table can be downloaded onto your hard drive to do further manual filtering with controls by using `scp path destination` (you can just download the whole `12_results` file, `-r` is a recursive function to repeat the command for all files). Navigate to your harddrive using `exit` and copy the  directory using the long form directory path.
```command
exit
Scp -r username@login.cac.queensu.ca:~/barque/12_results/ /Users/computername/Desktop/
````
We can now look at the species table in excel. Here we can filter out false negatives and positives using our controls.


## Making a backup directory to store old runs 
Its good practice to save your old run files and results so you can look back and see what changes you might have made. I do this by making a backup directory of the barque directory. For now you dont need to do this.
```command
git clone https://github.com/enormandeau/barque
#Rename directory to keep as backup by moving it to the new name `mv current_name new_name`.
mv barque 01_barque_backup
```
*note*: You can remove a lot of subdirectories from this to save space. Ie. Directories `05_trimmed` to `11_non_annotated` as well as the barque run file.

## Saving old runs
Copy all the log and results file to a unique folder in your backup directory.
```command
cp -r barque/12_results 01_barque_backup/12_results/UNIQUE_FOLDER_NAME
cp -r barque/99_logfiles 01_barque_backup/99_logfiles/SAME_AS_RESULTS_LOG
```
•	These two files have all the info you will need from your run whether successful or not. The results file will provide … your results. 
•	The log file has all the run information as well as your run parameters that you set in the `barque_config.sh` file so if you are troubleshooting these will show you where it failed and what parameters you were running.

## Cleaning barque after use 
You can run the below function while in the barque directory to remove all the data from each of the folders so you can run barque again without having to fully start over (ie. Copying in your dataset, database, configuration file…).
```command
./01_scripts/util/cleanup_analyses.sh
•	Not really recommended as Barque tends to not work after one use. You’re better off downloading a new Barque directory (see above).
```
# Formatting databases
Refer to the barque [README.md](https://github.com/enormandeau/barque/blob/master/README.md) file to format database to work in the barque pipeline.


