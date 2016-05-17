#Working with recombinant population

##Summary

During this hands-on session we will investigate the effect of recombination on genealogy reconstruction. We will use *hierBAPS* to perform genetic population structure analysis using a nested clustering approach and *Brat Next Gen* to identify recombination regions in the core phylogeny. We will then use the information provided by *Brat Next Gen* to mask the recombination regions from the core genome alignment. We will then use the new *unrecombined* alignment to infer genealogy by building a ML tree using FastTree.

##Population stucture using *hierBAPS*

###For *Windows* users

Using your sftp client (e.g. WinSCP or Filezillla) copy *core_gene_alignment.aln* and *core_gene_alignment.nwk* to your local computer.

*Note: please visit this site for a list of windows commands http://commandwindows.com/windows7-commands.htm.*
*Here some important ones: list your directory - dir; change the directory - cd; delete a file - del file; display a *.txt file - type file*

Go to "Start" and in the white box "Search programs and files" type *cmd*.
Click on *cmd.exe* to open the terminal.

In the terminal:

```
cd C:\<path\to\hierBAPS>
exData.exe <path\to\>core_gene_alignment.aln fasta
```

This will produce an input file called *seqs.mat* from you alignment file in the same directory of *hierBAPS*.

You now are ready to launch hierBAPS and perform the clustering. 
In the terminal type.

```
hierBAPS.exe seqs.mat <L> <maxK> results  
```

Clustering is performed with L levels in the hierarchy and maxK is the prior upper bound for number of clusters. As in BAPS, hierBAPS will estimate the maximum a posteriori partition (MAP) with the number of clusters in the interval 1 to maxK. hierBAPS will save an output file named results.mat (binary format) and a partition file "results.partition.txt", where each column represents the MAP partion of that layer.
As first you will run the analysis using L=2 and maxK=10. After the this first clustering you can check how the groups will be split at deeper levels. 
You can continue the clustering from the previous result to a deeper level (e.g. L=4) by typing

```
hierBAPS.exe results.mat 4 10 results2nd 
```

You can visualize the clustering by doing

```
drawSnpMat results.mat shuffle
```

This will draw a SNP matrix with rows and columns shuffled, and produce a tab delimited file "figInfo.txt".

Please note that the order in "results.partition.txt" is the same of the alignment and it is different from "figInfo.txt".

Now you can map the results on the core tree. Download the java version of TemPest from http://tree.bio.ed.ac.uk/software/tempest/ and extract the archive wherever. Open the *nwk file you generated using FastTree. Click "best-fitting root" and color the tips according to hierBAPS result, each cluster using a different color.

###For *Mac* and *Linux* users

Download the files you need using *sftp* commands

```
sftp <username>@taito.csc.fi
password

sftp> cd <path/to/roaryresults>
sftp> lcd <path/to/your/local/computer>
sftp> get <filename>
```
Open a terminal. Go to the directory containing the hierBAPS executable files and follow the instructions above.

###Comments on hierBAPS

*Note: the comment below, kindly provided by Prof. Jukka Corander, is a sort of summary of what is well explained in the original paper  http://www.ncbi.nlm.nih.gov/pmc/articles/PMC3670731/*

The program does a nested clustering of the population, i.e. given each cluster at the 1st level, the algorithm and the model seek statistical evidence for a further substructure within each cluster. If there is no significant signal of substructure within a cluster, it will not be further split in the 2nd level analysis. And so on, if even a 3rd level happens to be considered. The nested approach makes sense because most data sets are hierarchically structured and the deeper the branches between subclades, the less power one has to find fine level population structure in a single global analysis. At the finer (less deep in a tree) levels less strong recombination signals may become dominant again, when the polymorphisms only present in the other clades are pruned away in the hierBAPS model. In addition to finding monophyletic clades, the program can detect paraphyletic clades which are caused by recombination. Hence, by combining hierBAPS with a phylogenetic analysis one can see if there is evidence of recombination acting as a glue to some of the clusters.

##Recombination detection using *Brat Next Gen*

First you should check if the input sequences are evolutionarily close. Check if the median (+/- SD) of the root-to-tip distance in TemPest is not more than 0.1.

To run BNG please use the instructions here:
http://users.ics.aalto.fi/pemartti/win32_brat/BratNextGen_manual.pdf

*Note: the built-in method for estimating alpha is optimized for bigger samples and, sometimes BNG fails when there are few genomes. In our case we have too few genomes, and we need to fix value of alpha. With "normal" dataset that is not needed if alpha is less than 20. If it is higher than 20, fix it at 20. Marttinen Pekka recommended alpha=20 as a default solution to use as manual input, if the automated learning does not work. The value is based on empirical knowledge gained from experiments with multiple data sets, so not really a theoretical justification. The very large value corresponds to assuming that the allele frequencies are exactly 0.5,0.5 (in case of two possible alleles), which of course we can discard as not reasonable.*
*There will be an updated version of BNG that will be made public in summer. Once that's out, I recommend to have a try with that.*

1. "File" => load Data => *core_gene_alignment.aln*
2. Set alpha=1 in "Option" => Set hyperparameter
3. Click "Draw PSA tree".  Set a cutoff in the PSA tree. Use the hierBAPS clustering results to guide your selection, i.e. keep the same number of clusters.
4. Click "learn recombinations", set the number of iterations to 5.
5. Click "estimate significance", click "on this computer", set number of replicate runs to 10, and choose the output directory.
6. Set significance limit to 0.05.
7. Click "Results" => "Draw Segments".
8. Click "Results" => "Write tree leaf names" to get the leaf names.
9. Click "Results" => "Write Segments". This will provide you the recombination fragments file.

*Note: Roary orders the gene alphabetically in the coregenome alignment. For the model implemented in BNG, it's in general better if the core is more contiguously ordered. However it did not harm in this paper http://gbe.oxfordjournals.org/content/5/8/1524.abstract?sid=232d967a-411b-45f9-9747-60dd2f83e4d4*


##Mask recombination events and build a genealogy tree

Upload to your csc account the *Brat Next Gen* tabular files using sftp.

Mask the recombination events using the perl script *exclude_recombination.pl* available in the directory *scripts*.

```
module load biokit
perl <path/to/scripts>/exclude_recombination.pl <path/to/tabular_BNG_file> <path/to/core_gene_alignment.aln>
```

The script produced a file named *.screened.fas. 

*Note: unfortunately the script has a bag and before running FastTree the first line should be removed*

Remove the first line of the *.screened.fas.

```
sed '1d' <path/to/*.screened.fas> > <path/to/newfile.fas>
```

Run Fasttree

```
screen
sinteractive
module load biokit
[path/to/]FastTree -nt < [path/to/newfile.fas] > core_gene_unrecom.nwk
exit
```
**NOTE:** Remember with *Ctrl a d* you can always close the *screen* session. You can reattach the *screen* by typing *screen -r job title*

After ~10 min check if the job is finished 

Now you can map the hierBAPS results on the core tree. Download the *core_gene_unrecom.nwk* to your local computer using sftp. Open the *nwk file with TemPest. Click "best-fitting root" and color the tips according to hierBAPS result, each cluster using a different color.

#Compare the core tree vs genealogy

For comparing two trees we will use a script developed by by Morgan N. Price in Adam Arkin's group at Lawrence Berkeley National Lab.
You can find the perl script in the directory *scripts*.
For interpreting the results please check http://meta.microbesonline.org/fasttree/treecmp.html

```
perl <path/directory/scripts>/CompareTree.pl -tree treeFile1 -versus treeFile2
```




