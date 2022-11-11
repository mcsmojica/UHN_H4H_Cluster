# UHN_H4H_Cluster

Getting started with the UHN Cluster

## Sections:
* [Basic Overview of the cluster](#the-lay-of-the-cluster-land)
* [Reporting Issues with the Cluster](#issues-with-the-cluster)
* [Step 1a) Logging on to the HPC](#step-1a-logging-on-to-the-hpc)
* [Step 1b) Understanding the usage of the login vs data transfer nodes](#step-1b-understanding-the-usage-of-the-login-vs-data-transfer-nodes)
* [Step 2) Setting up your .bashrc](#step-2-setting-up-your-bashrc)
* [Step 3) Transferring Files in and Out of the Cluster](#step-3-transferring-files-in-and-out-of-the-cluster)
* [Basic overview of Slurm](#slurm)
* [Step 4a) Submitting Jobs to Slurm](#step-4a-submitting-jobs-to-slurm)
* [Step 4b) Using Programs on the Cluster](#step-4b-using-programs-on-the-cluster)
* [Basic Slurm Commands](#basic-slurm-commands)
* [Using GPU Nodes](#using-gpu-nodes)

## The lay of the cluster land
You can think of the cluster as thousands of computers, each individual computer being considered a “node”. For our purposes / ease of understanding, the UHN cluster has 3 main types of nodes: the login node, the data transfer node, and the compute nodes. The compute nodes can be either CPU nodes or GPU nodes (explained further below). The nodes are grouped into different partitions, which have different maximum levels of resources allocated to them. 
![cluster_overview](https://github.com/mcsmojica/UHN_H4H_Cluster/blob/main/clusterland.png?raw=true)

In order to manage who gets to use what compute nodes, for how long, and to what extent of the compute node’s resources, the cluster uses a prettily named program called [Slurm](https://slurm.schedmd.com/) that manages everything for us. 
The only way to use the compute nodes is to issue commands to Slurm which will assign your job to a queue and eventually to a node (or several, if you are allowed to for your job type) – otherwise, the compute nodes are inaccessible. Your job’s priority is determined by your group’s target usage, your account’s historical resource utilization, time that you job has stayed in the queue, partition that your job will be launched, and resources you request. 
For data security / compute management purposes, there are also some quirks in terms of how the login node and data transfer node are used (described here).
The HPC cluster is a shared resource- thus, only request resources (cpu, memory, walltime) needed for your jobs and please clean up your intermediate results as soon as possible.
## Issues with the Cluster
If you have any issues on the cluster, please send email to hpc_support[at]uhnresearch.ca. A ticket will be created for the issue. 
## Step 1a) Logging on to the HPC
After getting an account with your research ID or TID (contact hpc_support[at]uhnresearch.ca for info on how to do this), you can ssh onto the cluster with either Global Protect VPN on or working on a lab computer with direct intranet access in two different ways:
1)	Logging on to the login node: ssh -XY username@h4huhnlogin1.uhnresearch.ca
2)	Logging on to the data transfer node: ssh -XY username@h4huhndata1.uhnresearch.ca
### Navigating Around the Directory Tree within the Cluster
After you log in, there are basically only two main directories that you need to know about; your own home folder, and your group folder. You can navigate in the terminal to both your home folder and the groupname folder from the data transfer node when you ssh into it (h4huhndata1.uhnresearch.ca); but you can navigate only to your home folder in the login node (h4huhnlogin1.uhnresearch.ca)
![directory_tree](https://github.com/mcsmojica/UHN_H4H_Cluster/blob/main/directory_tree.png?raw=true)

### Purpose of the Home / Groupname directories
#### User Home folder – 50GB
•	Mainly for storing config files
•	Some jobs may write things here – monitor space here to make sure it doesn’t fill up
#### Group folder – 2TB (if free tier; can be 10TB+ in size depending on the space purchased by your lab)
•	/cluster/projects/groupname should be used as the working directory for your processes since it has the most space
•	Transfer your data into here, output processed files here from your slurm jobs, set your tmp folder to be here in your .bashrc (see below)
•	Note that if you have multiple HPC users in your lab, this directory is a shared space for everyone
## Step 1b) Understanding the usage of the login vs data transfer nodes
There are a few different things to consider when deciding which node you want to log on to (you can also log into both nodes at the same time in separate terminals, which is helpful when you need to do different tasks at once):
![nodes](https://github.com/mcsmojica/UHN_H4H_Cluster/blob/main/nodes.png?raw=true)
In short…:
1) Sign on to the login node when you are ready to run slurm jobs. 
2) When you want to transfer data, you will want to sign on to the data transfer node.
3) It’s also a bit easier to edit scripts / move around in the data transfer node, since you have access to both your home folder and the groupname folder whereas you only have access to your home folder on the login node. So, sign on to the data transfer node when you want to edit your scripts on the cluster / transfer scripts to the cluster
## Step 2) Setting up your .bashrc
After logging in, copy paste the following lines of code into your .bashrc file on the cluster:
### Change command prompt (optional, but recommended)
```
thiscom=$HOSTNAME
## PS1 (aka command prompt) setup
# Colour codes
txtcyn='\[\e[0;96m\]' # Cyan
txtpur='\[\e[0;35m\]' # Purple
txtwht='\[\e[0;37m\]' # White
txtlbl='\[\e[1;34m\]' # Light Blue
txtred='\[\e[0;31m\]' # Red
txtrst='\[\e[0m\]'    # Text reset
txtora='\[\e[38;5;214m\]'  #orange; see https://misc.flogisoft.com/bash/tip_colors_and_formatting
#Set colours for prompt parts
# note: $thiscom is from the .bashrc; uses nickname
userC="${txtwht}"
hostC="${txtcyn}"
timeC="${txtpur}"
pointerC="${txtlcyn}"
pathC="${txtora}"
termC="${txtrst}"

#\u  username   \@ time 12hr am/pm   \$   changes if root#
#Build the prompt
export PS1="${userC}\u ${hostC}$thiscom ${pathC}\w  ${timeC}\@ ${termC}\$  "
## Custom LS_COLORS so it doesn't hurt ur eyes with your pwetty custom gnome-term theme
export LS_COLORS=$LS_COLORS:"di=0;96":"fi=0;97":"ex=0;93"
```

### Set tmp directory environment variables (mandatory)
```
export TMP=/cluster/projects/groupname/tmp/
export TMPDIR=/cluster/projects/groupname/tmp/
export TEMP=/cluster/projects/groupname/tmp/
```

### Set default file permissions (recommended)
```
#permissions
umask 007
```

### Create bash function to quickly cd to group folder (recommended)
```
cdgroup () {
cd /cluster/projects/groupname
}
```

### Create variable for your work computer IP addresses (see here for usage explanation):
```
#ip addresses
yourcomp=”###.##.##.##”
```

### Set Paths for Module Programs (update the versions as needed. Below are examples for commonly used neuroimaging programs)
```
#FSL
FSLDIR=/cluster/tools/software/centos7/fsl/6.0.5.2
. ${FSLDIR}/etc/fslconf/fsl.sh
PATH=${FSLDIR}/bin:${PATH}
export FSLDIR PATH
#mrtrix3
export PATH=/cluster/tools/software/centos7/mrtrix3/3.0.3/bin:$PATH
#freesurfer
export FREESURFER_HOME=/cluster/tools/software/centos7/freesurfer/7.3.2/
#ANTS
export ANTSPATH=/cluster/tools/software/centos7/ANTs/2.4.2/bin/
export PATH=${ANTSPATH}:$PATH
export ITK_GLOBAL_DEFAULT_NUMBER_OF_THREADS=1
```

## Step 3) Transferring Files in and Out of the Cluster
Please be mindful of how much space you will need / how much space is available / how much space you have used when using the cluster. If your directory fills up, things will grind to a halt until space is freed up. Thus, transfer your unneeded files off the cluster ASAP.
### Transferring files INTO the Cluster
You can transfer files into the cluster when you are either signed on a computer that has VPN on, or are using computers that are on UHN’s network. You can only transfer data into the data transfer node.
To do so, use rsync from your computer to transfer data to the /cluster/projects/groupname directory, and scripts to either your group directory: /cluster/projects/groupname if they are scripts you want to run on your data, or to your home folder /cluster/home/username if they are sbatch run scripts (discussed below).
Note: the -R flag for rsync means transfer the directory structure from the source as is. This is useful when your scripts are already set up for a particular directory structure for analyses.
EXAMPLES (data transfer):
```
rsync -Rahv /data/Ariana_Data_on_compname username@h4huhndata1.uhnresearch.ca:/cluster/projects/groupname/data
rsync -Rahv /data/Ariana_Data_on_compname --include="*/" --include={"*.mif","*.txt","*.bvec","*.bval","*.nii.gz"} --exclude="*" username@h4huhndata1.uhnresearch.ca:/cluster/projects/groupname/data
#the above command can be used to transfer specific file extensions only and exclude all others. Handy to save space
```
### Transferring files OUT of the Cluster
You can use rsync commands to also transfer data off of the cluster using the data transfer node from your group or your home folder.
However, the cluster will not recognize computer hostnames – it can only recognize IP addresses as being part of the UHN network and that have approved status from HPC (may need to send in a ticket to have your ip address recognized). Only IP addresses that are part of the UHN network can have data transferred to them from a terminal on the cluster (you can alternatively rsync from a terminal on your machine rather than a terminal on the cluster if this doesn’t work for you because your PC is not on the internal network). You will need to use the IP address for your work PC instead of your hostname in your rsync command – to make this simpler, create a variable in your .bashrc file with your computer’s IP address (see here). Now instead of writing out your IP address each time, you can instead refer to this variable by for example, inputting $yourcomp on the terminal. 
EXAMPLE (while signed on to the data cluster node, this command will transfer the file run_sbatch.sh from the group script directory to the desktop of username on yourcomp):
```
rsync -rahv /cluster/projects/groupname/scripts/run_sbatch.sh username@$yourcomp:/home/username/Desktop
#notice the “$” sign infront of yourcomp, which signifies it as a bash variable.
```

## Slurm 
Once you have your data on the cluster, to actually use the cluster’s resources to do work on it, you need to submit jobs to Slurm. You can only submit jobs when you are logged on to the login node.
When a user submits a job, Slurm will schedule this job on a node (or nodes) that meets the resource requested by the user. If no resources are currently available, the user’s job will wait in a queue until the resources they have requested become available for use. This is why only requesting the resources you need is beneficial to you – the less resources your request, the faster your job will go through the queue to be actually run as it won’t have to wait as long for more nodes to free up if it needs a lot of resources.
### Partitions
Nodes are divided into distinct partitions based on the resources allocated to them (some nodes are part of the high memory partition, others part of the gpu partition, etc.) Different partitions have different default and maximum resources available to them. To set the partition you wish to use, you will use (slurm commands explained more here):
```
#SBATCH -p <partition-name>   #if using a sbatch script
sbatch -p <partition-name>   #if writing sbatch command directly into command line
```
Please use the appropriate partition for the amount of resources you require or they may be killed by the scheduler. You can check the up-to-date partition list using “sinfo”. Below is a table showing the main partitions available and their resources
![partitions](https://github.com/mcsmojica/UHN_H4H_Cluster/blob/main/partitions.png?raw=true)

** while you launch slurm jobs from the login node, do not run the slurm jobs ON the login node itself when specifying the partition you want to use
## Step 4a) Submitting Jobs to Slurm
There are two main kinds of jobs you can submit to Slurm – interactive jobs, and non-interactive jobs.
### Submitting an Interactive Job to Slurm
When you want to be able to work interactively with programs on the cluster in a terminal, you can use the command “salloc” to request an interactive job. Note that interactive jobs are limited to 10 hours, so you won’t want to leave something running for too long on an interactive job or it will be killed and will not complete. Interactive jobs should instead be used for testing/ troubleshooting / small tasks.
Example:
```
salloc -c 1 -t 1:0:0 --mem 1G
```
### Non-interactive jobs – submitting scripts for Slurm to run
You can issue commands to Slurm directly in the command line, but for repeat use, the best way to issue Slurm commands is using a script with lines of #SBATCH options. For example, let’s say the following is saved in file run_sbatch.sh: 
```
#!/bin/bash
#SBATCH -t 1-0:00:00     # walltime for your job in format of days-hours:minutes:seconds
#SBATCH --mem=30G     # amount of memory your job needs
#SBATCH -J KET01     # name of your job
#SBATCH -p all     # partition you job wants to run
#SBATCH -c 1     # number of CPUs your job needs to run
#SBATCH -N 1     # number of nodes your job needs to run. Please use 1 node unless you are running mpi jobs.
#SBATCH --error=/cluster/projects/groupname/errors/%u.%x.errors   # make errors go to /errors
#SBATCH --output=/cluster/projects/groupname/output/%u.%x.output   # output go into /output
#SBATCH --mail-user=username@uhnresearch.ca  #mail to : address
#SBATCH --mail-type=ALL   #get mail for all kinds of updates to job

**Your commands here**
```
This way you can set the options for sbatch resource allotment inside the script run_sbatch.sh for your job to run. To submit the job and run the commands written after the #SBATCH options, you simply type in the terminal on the login node:
```
sbatch /path/to/run_sbatch.sh
```
Instead of writing out lines of commands you want to be run after the #SBATCH options, a neater way is to use run_sbatch.sh to launch scripts you have saved in another file. For example, let’s say you have a script saved as MRtrix-analysis.sh which contains a bunch of commands to run an analysis. You would edit run_sbatch.sh like so:
```
#!/bin/bash
#SBATCH -t 1-0:00:00     # walltime for your job in format of days-hours:minutes:seconds
#SBATCH --mem=30G     # amount of memory your job needs
#SBATCH -J KET01     # name of your job
#SBATCH -p all     # partition you job wants to run
#SBATCH -c 1     # number of CPUs your job needs to run
#SBATCH -N 1     # number of nodes your job needs to run. Please use 1 node unless you are running mpi jobs.
#SBATCH --error=/cluster/projects/groupname/errors/%u.%x.errors   # make errors go to /errors
#SBATCH --output=/cluster/projects/groupname/output/%u.%x.output   # output go into /output
#SBATCH --mail-user=username@uhnresearch.ca  #mail to : address
#SBATCH --mail-type=ALL   #get mail for all kinds of updates to job

/path/to/MRtrix-analysis.sh
```
Now when you type sbatch /path/to/run_sbatch.sh in the terminal, it will start a job with the options you’ve specified (in this case, 1 day walltime, 30G memory, job name KET01, on partition all, 1 CPU, 1 node, etc.) and it will run your script, MRtrix-analysis.sh. In this way, you can simply edit the run_sbatch.sh file with different scripts / options for sbatch,  and then have separate scripts you refer to in run_sbatch.sh for your actual analyses. In short, you have an sbatch launch script, and then you have your analyses scripts this launch script will run. 
Because we can only launch slurm commands from the login shell, and the login shell only has access to your home directory, run_sbatch.sh will need to be saved somewhere in your home directory so that you can use it for sbatch. Since once slurm is run, you gain access to your group folder also, the other scripts can be located within the group directory.
### Run a single command on the cluster
You can use the command srun to run a single command on the cluster.
#### srun 
Run a single command on the cluster.
```
srun <your command>
```
Example:
```
srun -c 1 -t 1:0 --mem 128M hostname
```

## Multistep Jobs / Arrays 
TBD
## Step 4b) Using Programs on the Cluster
Programs are accessible for your use either through the program-management application on the cluster called module, or through lab-made containers that have custom configurations of particular programs installed within the container for use.
### Using Module programs on the cluster
The HPC cluster uses “module” to manage software on the cluster. 
To see what modules/software is available for use, use the command:
```
module avai
```

This will list all available software and all the available versions of the software.
If there is software missing you need for your work, you can request it to be installed by emailing: hpc_support@uhnresearch.ca
To view details of a specific module, use the command:
```
module show <module/version> 
```
To use software from module, you need to load it to your environment first. You do so with the command:
```
module load <module/version> 
```
To remove software from your environment, use the command:
```
module unload <module/version> 
```
If you need to use these programs in your jobs, note that you will need to include module load commands in your script to load the required modules for your script to have access to these programs.
### Using Singularity/Apptainer Containers on the cluster
In the event that the HPC staff are unable to install modules you need, or you otherwise have more specific set-up requirements / want more control over your programs, you can create a container outside of the cluster with all the programs you need and then rsync that container into the cluster for use. You should put the container under your home directory to save space on our group directory, though both would technically work. Singularity version 3.5 is available on the cluster and can be loaded using module to use singularity commands to run your container. An example script to launch a script called em_script.sh inside the container DavisHPC_container.sif is below:
```
#!/bin/bash

#SBATCH -t 1-0:00:00     # walltime for your job in format of days-hours:minutes:seconds
#SBATCH --mem=30G     # amount of memory your job needs
#SBATCH -J KET01     # name of your job
#SBATCH -p all     # partition you job wants to run
#SBATCH -c 1     # number of CPUs your job needs to run
#SBATCH -N 1     # number of nodes your job needs to run. Please use 1 node unless you are running mpi jobs.
#SBATCH --error=/cluster/projects/groupname/errors/%u.%x.errors   # make errors go to /errors
#SBATCH --output=/cluster/projects/groupname/output/%u.%x.output   # output go into /output
#SBATCH --mail-user=username@uhnresearch.ca
#SBATCH --mail-type=ALL
module load singularity
singularity exec --containall --bind /cluster/projects/groupname/tmp:/tmp --bind /cluster/projects/groupname/data:/data --bind /cluster/projects/groupname/output:/output --bind /cluster/projects/groupname/errors:/errors --bind /cluster/projects/groupname/scripts:/scripts /cluster/home/username/DavisHPC_container.sif bash /scripts/em_script.sh
```

Note that because the container has bound the /cluster/projects/groupname/scripts folder to /scripts inside the container, and the bash command is being run inside the container, the path to the script em_script.sh is just /scripts, as this is where the container can find it.
## Basic Slurm Commands
### sinfo 
This command is to check the status of the cluster/partitions 
```
sinfo 
sinfo –lN    # same as above, but shows per-node status
```
### squeue 
This command is to show the status of jobs 
```
squeue 
squeue –l  # long format
```
### scancel 
This command is to kill jobs 
```
scancel <jobID>         # kill job with <jobID>. 
scancel -u <username>   # kill all jobs for user <username>. 
scancel -t <state>      # kill all jobs in state <state>. <state> can be PENDING, RUNNING, SUSPENDED
```
### sshare 
This command is to show Slurm share information 
```
sshare 
sshare –l  # long format
```
## Using GPU Nodes
(thanks @scarere , @briannghiem , and @ nbwuzhe)

<!--  Ariana: To use the GPU nodes, you need to use your “gpu” account and one of the “gpu” partitions. You also need to request the number of GPUs you need to use. For example,
```
sbatch  -A groupname_gpu -p gpu --gres=gpu:1 my_script.sh
```  -->

If you wish to use GPUs, make sure to select the GPU partition when allocating resources. You can do so by adding the argument '-p gpu' to your salloc command or job script. You also have to provide the account for your lab:
```
salloc -p gpu --account=groupname_gpu -t 3:00:00 -c 6 --mem 20G
```
Note that the --mem flag above refers to system/cpu memory (RAM) and NOT the GPU memory (VRAM).

The 'gres' flag is used
- If you wish to select multiple GPUs. For example,
    ```
    salloc -p gpu --account=groupname_gpu --gres=gpu:3 -t 3:00:00 -c 4 --mem 20G
    ```
- To specify the GPU of choice. There are two kinds of GPUs on the UHN H4H cluster.
    1. [Tesla V100 PCIE](https://www.nvidia.com/en-us/data-center/v100/)
        * denoted by 'v100'
        * 16 or 32GB VRAM
    Note that simply adding the 'v100' argument means that you could be allocated the 16GB version. If you need the 32GB v100's, then you need to add the -C gpu32g flag when allocating resources:
    ```
    salloc -p gpu --account=groupname_gpu --gres=gpu:v100:3 -t 3:00:00 -c 4 --mem 20G -C gpu32g
    ```
    2. [Tesla P100 PCIE](https://www.nvidia.com/en-us/data-center/tesla-p100/)
        * 12GB VRAM
        * denoted by 'p100'
    ```
    salloc -p gpu --account=groupname_gpu --gres=gpu:p100:1 -t 3:00:00 -c 4 --mem 20G
    ```