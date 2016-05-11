#Group work

##Summary

This file contain the guidelines for running the tasks to perform in a group.

You might decide to run only one job for group or each member of the group will run a job. However, be aware that you are alone in this activity and if you need help you need to find it within your group.
This task is propaedeutic to the afternoon activities and aims to give you the chance to test the skills you learnt in the morning.   
You will annotate *Campylobacter jejuni* genomes using *Prokka* (it would take approximately 45 min) and then extract core and accessory genome list using *Roary* (it would take approximately 30 min). Using he *core genome alignment* extract by *Roary* you will build a tree using *FastTree*. These analyses serve for investigating recombination events using *Brat Next Gen* and inferring population structure using *hierBAPS*.


##Modify the batch script and run Prokka

Go to the directory *scripts*.
Open the file *Prokkabatch.sh* in *nano*:

```
nano Prokkabatch.sh
```
and change the following lines:

```
#SBATCH --mail-user= [put your E-mail address]
cd [path/to/Campylobacter/small_dataset]
```

Run the batch script:

```
sbatch Prokkabatch.sh
```

##Run Roary
You will run Roary in interactive mode. 
Open a Taito-shell session:

```
sinteractive
```

Load the modules you need.

```
module load biokit
module load prokka
```

Go to the directory containing the folder *prokka-gff*.
Build a directory named *roary*.

```
mkdir roary
```

Run Roary.

```
roary -e --mafft -o small_dataset_proteins -f roary -cd 100 -p 16 *.gff
```

##Run FastTree

You installed FastTree in your appl_taito directory. To run it you must call it from this directory. You need to run FastTree using the core_gene_alignment.aln built buy Roary.

Run FastTree.
*Note: if the directory where FastTree is different be sure to call it correctly. I'm assuming you are still in taito-shell.*

´´´
$HOME/appl_taito/FastTree -nt < core_gene_alignment.aln > core_gene_alignment.nwk

