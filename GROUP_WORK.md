#Group work

##Summary

Herein you can find the guidelines for running the tasks to perform in a group.

You might decide to run only one job for group or each member of the group will run a job. However, be aware that you are alone in this activity and if you need help you need to find it within your group.
This task is propaedeutic to the afternoon activities and aims to give you the chance to test the skills you learnt in the morning.   
You will annotate *Campylobacter jejuni* genomes using *Prokka* (it would take approximately 45 min) and then extract core and accessory genome list using *Roary* (it would take approximately 25 min). Using he *core genome alignment* extract by *Roary* you will build a tree using *FastTree* (it would take approximately 10 min). These analyses serve for investigating recombination events using *Brat Next Gen* and inferring population structure using *hierBAPS*.


##Modify the batch script and run Prokka

Go to the directory *scripts*.
Open the file *Prokkabatch.sh* in *nano*

```
nano Prokkabatch.sh
```
and change the text between **[bracket]**:

```
nano Prokkabatch.sh

    #!/bin/bash -l
    # created: May 10, 2016 6:08 PM
    # author: **[useraccount]**
    #SBATCH --constraint="snb|hsw"
    #SBATCH -o stdO
    #SBATCH -e stdE
    #SBATCH -p serial
    #SBATCH -n 16
    #SBATCH -t 01:00:00
    #SBATCH --mail-type=END
    #SBATCH --mail-user= **[your Email]**

    # commands to manage the batch script
    #   submission command
    #     sbatch script-file
    #   status command
    #     squeue -u your useraccount
    #   termination command
    #     scancel jobid

    # For more information
    #   man sbatch
    #   more examples in Taito guide in
    #   http://research.csc.fi/taito-user-guide

    #running prokka
    module purge
    module load biokit
    module load prokka
    cd **[path/to/Campylobacter/small_dataset]**
    for file in *.fasta; do prokka --cpus 16 --outdir $file-dir --prefix $file- $file; done
    mkdir prokka-gff
    for dir in *dir; do mv $dir/*.gff ./prokka-gff; done

    # This script will print some usage statistics to the
    # end of file: stdO
    # Use that to improve your resource request estimate
    # on later jobs.
    used_slurm_resources.bash

```

Run the batch script

```
sbatch Prokkabatch.sh
```

##Run Roary

Open a new session in Taito

```
sinteractive
screen -S <name your job>
```

Load the modules you need.

```
module load biokit
module load prokka
```

Go to the directory containing the folder *prokka-gff*.
Build a directory named *roary*.

Run Roary.

```
roary -e --mafft -o small_dataset_proteins -f roary -cd 100 -p 4 [path/to/prokkaGFF/files]
```

exit from the session *Ctrl a d*

After ~25 min check if the job is finished 

```
screen -r <job name>
```

##Run FastTree

Open a new session in Taito

```
sinteractive
screen -S <name your job>
```

Load the modules you need.

```
module load biokit
```

You need to run FastTree using the *core_gene_alignment.aln* built by *Roary*.

```
[path/to/]FastTree -nt < [path/to/core_gene_alignment.aln] > core_gene_alignment.nwk
```
exit from the session

After ~10 min check if the job is finished 

exit from the session *Ctrl a d*

After ~15 min check if the job is finished 

```
screen -r <job name>
```

