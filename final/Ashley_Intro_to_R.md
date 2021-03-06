#Ashley's Bonus Quick-R Tutorial for comparative diversity
***
Authored by Ashley Shade   
[EDAMAME-2015 wiki](https://github.com/edamame-course/2015-tutorials/wiki)

***
EDAMAME tutorials have a CC-BY [license](https://github.com/edamame-course/2015-tutorials/blob/master/LICENSE.md). _Share, adapt, and attribute please!_
***

##Overarching Goal  
* This tutorial will contribute towards understanding **ecological statistics to analyze and interpret microbial sequencing data**

##Learning Objectives
* Read a traditional OTU table and a resemblance matrix into R 
* Use the vegan package to calculate resemblances
* Visualize comparative diversity using heatmaps and ordinations
* Export tables from R.

***
**NOTE:  THIS TUTORIAL IS A WORK IN PROGRESS**

###1. Fast Introduction to R: formatting files and reading in tables 
Create a new directory for R analyses, above the Manduca_raw_data directory. This is helpful to keep analyses separate, and to not accidentally alter the output files from QIIME that you may be using repeatedly for various analyses. copies into the new directory: 

The classic OTU table: 
otu_table_even.txt

The mapping file: 
map.txt 

The phylogenetic resemblance matrices: 
Make copies of the following files, and move them to your R working directory. 
Open each of these files in Excel and sort them by SAMPLE IDs so that every sample is in the same order. You will have to sort both rows and columns for the resemblance matrices. Also, if the samples IDs do not have the same character length, they will not be in consecutive order. This is okay, as long as all of the samples are in the same order in each file.  

Hints for sorting a resemblance matrix in Excel: 
1.	Sort by Rows. Highlight all of the data EXCEPT for the first column (which contains row  IDs). From the excel Menu, select Data -> Sort. Click on the Bottom Left button "Sort Options." Select "sort left to right." Select OK. Select "Sort by Row 1". Click okay. Cells should be sorted left to right.   
2.	Sort by Columns. Highlight all of the data. From the Excel Menu, select Data -> Sort. Click on the bottom left button "Sort options". Select "Sort top to bottom". Select OK. At the bottom of the box select "Header row." At the top of the box select "(Column A"). Select OK. Cells should be sorted from top to bottom.   
3.	Sanity check:  Check that the diagonal is ZERO.  

Open R, and if you haven't already, install the vegan package for community ecology. On a Mac, this is down by going to the menu bar, Tools -> Install packages. Select the box to also install dependencies (dependencies = existing packages/functions that vegan needs to execute some of its functions) 
Note: Here is a [link](http://vegan.r-forge.r-project.org/) to info about the vegan package. 
Then, to load the functions of the vegan package into your current R environment, in the R console and type:
```
library(vegan)
```

Move into the  directory to the one that was just created. On the Mac menu bar, select Tools -> Set working directory -> Choose directory. 
Or, if you are using R Studio, navigate to the lower right panel, click on the "Files" tab, and the click on the top right-hand side "..." button (below the refresh arrow, see below), to browse to the correct directory. Then click on the "More" tab, and select "set as working directory." 
Or, you could use the "setwd()" command in the console. Lots of options!   
Now, we will use our OTU table to calculate two resemblance matrices using functions from the vegan package.   
First, we must read in the OTU table. Before R will accept the OTU table, we must do a little bit of formatting. Open the tab-delimited file in Excel (or, if you are using R Studio, you can open it there), and clear the "#OTU ID" cell. Then, save the file (make sure it is a tab-delimited txt file). The "#" tells R to skip reading the row, but this would remove our sample names and cause an error with analysis. Then, we will read the table into R using the `read.table()` command. We name the table "otu" : this makes it easy to call in R.   This table should have an even number of sequences per observation (evenly-subsampled); otherwise, community comparisons are invalid because of differences in sampling effort across observations.    
```
otu=read.table("data_even.txt", header=TRUE, row.names=1, sep="\t", check.names=FALSE)
```

Then, use the `head()` command to view the top of otu. Check that the sample IDs, and the OTU IDs reasonable (sanity check!), and that the formatting seems okay:
```
head(otu)
```

Now, let's explore working with tables. In R, we use brackets `[rows, columns]` to specify rows and columns within a table. What is the value in your OTU table that is in the 19th row and 23 column? 
```
otu[19,23]
```

You can also use colons to specify ranges, as in the example below, where we've selected the first five rows and the samples in columns 5 through 10. 
```
otu[1:5, 5:10]
```
We can also extract row names (OTU IDs) and column names (sample IDs) using R
```
row.names(otu)

colnames(otu)
```
We can also use the sample or OTU IDs to explore values in the OTU table. Look at the sample called KM41 by calling this column name in quotations:
```
otu[,"ColumnName"]
```
Look at OTU 134 by calling this row name in quotations: 
```
otu["134",]
```
Finally, we can check the dimensions of the table, to make sure that it is the size (number of OTUs and number of observations/samples) we expect: 
```
dim(otu)
```
Get into the habit of checking your tables to make sure they match your expectations. Let's start with the columns. This should be the number of samples PLUS one column that has the taxonomic IDs (labeled "ConsensusLineage").  What about the rows?
Note: If for any reason your expectations of your own dataset are not met, check formatting of the input files. In my experience, formatting issues are the number one reason for errors.   
Read in the mapping file and use the "attach" function to link the information to the OTU table. But, there is some formatting that we must do first. First, open the file and remove the "#" on the first line, but maintain the column name "SampleID". The “#” indicates to R to ignore the row, which is needed in QIIME but NOT needed in R because we want to use the column names to sort our contextual data.   
Second, ensure the samples are in the same order across files. The R analyses described below only work if the samples are in the same order across files! 
Third, remember that there was one sample, KM43 that had low sequences and was omitted from the final OTU table. We need to omit it from the map file also.   
```
map=read.table("Manduca_map_sorted.txt", header=TRUE, sep="\t")
```

For the next step, we will remove the ConsensusLineage labels from the table, as we do not need them at the moment. We do this by first calling the labels as a vector, rdp, and then removing the whole column (column 77) from the table. Now, we have a vector of taxonomic assignments called rdp, plus a "classic" sparse OTU table, called otu. You can use your R workspace browser (upper right window in R studio) to keep track of the objects you have in your R environment, and their dimensions.   
``` 
rdp=otu[,ncol(otu]
names(rdp)=row.names(otu)
otu=otu[,-ncol(out)]
dim(otu)
```

###2. Comparative (beta) diversity in R: making, reading in, and comparing resemblance matrices 
We will use the vegdist function to calculate a Bray-Curtis and a Sørenson resemblance matrix from the OTU table. Navigate to the help window, and type in `vegdist()` to see the arguments for the function. We first need to transpose the OTU table using the `t()` command to have species in columns and samples in rows, just a formatting difference for the function - no big deal. It is easy to transpose the table directly inside the vegdist command. 
```
braycurtis.d=vegdist(t(otu), method="bray")

head(braycurtis.d)
```

Now 'braycurtis.d' is the name of our new resemblance matrix. I use the .d ending to remind myself that it is a resemblance matrix. R sees a resemblance matrix as something different than a table or data frame, as it sees the otu table. Instead, R sees a very long vector. That is why the 'head()' command returns the first few values of a vector instead of a table with rows and columns.   
Sorenson's similarity has the exact same calculations as Bray-Curtis, except that it does not weight species. Therefore, we make a Sorenson matrix using the exact same protocol, except that we set the binary argument to TRUE to tell R not to use the information about the taxa relative abundances in the OTU table. Compare the head of sorenson.d to that of braycurtis.d. 
Note: Bray-Curtis can be calculated as a similarity or dissimilarity metric, but it is traditionally used as a dissimilarity, as it is in the vegan package. As an example, it could be that  two samples  are 0.19 (%) dissimilar (or, 0.81 similar) using Bray-Curtis, but can be 0.45 dissimilar (0.55 similar) using Sørenson. This is typical, as not using information about the relative contributions of taxa may make patterns become less apparent. UniFrac is also a metric of distance, so the same logic applies for their interpretation. 
```
sorenson.d=vegdist(t(otu), method="bray", binary=TRUE) 
head(sorenson.d)  
```
   
Now, we read in the UniFrac resemblance matrices made in QIIME. When we read them into the R environment, R recognizes them as tables. We use the `as.dist()` command to let R know that these are resemblance matrices. Run the command lines one at a time, and inspect the head of each object as you go along. We name the new resemblance matrices “weighted_u.d” and “unweighted_u.d”

```
weighted_u=read.table("weighted_unifrac_Manduca_otu_table_even861_sorted.txt",
header=TRUE, row.names=1, sep="\t")
head(weighted_u)
weighted_u.d=as.dist(weighted_u)
head(weighted_u.d) unweighted_u=read.table("unweighted_unifrac_Manduca_otu_table_even861_sorted.txt",
header=TRUE, row.names=1, sep="\t")
head(unweighted_u)
unweighted_u.d=as.dist(unweighted_u)
head(unweighted_u.d)
```

###4. Visualizing community data in R 
####4.1 Heatmaps + clustering 
It is quick and easy to generate a heatmap from an otu table. First, install the "gplots" package in the same way that you installed vegan. Don't forget the dependencies. We will set colors to range from white to black, using the `colorRampPalette()` command with twenty steps between the minimum and the maximum values. Then, we call heatmap.2(), again transpose the OTU table, and, set the argument trace = "none" so that obnoxious turquoise lines don't show up in the final plot. 
```
library(gplots)
colors <- colorRampPalette(c("white", "black"))(20)
heatmap.2(t(otu), trace="none",col=colors)
```

Inspect the plot. Zoom in if you can, as it may be hard to distinguish the sample IDs and the OTU IDs because the dataset is large. The default heatmap.2 algorithm clusters samples that have similar compositions, and also OTUs that have similar abundances and occurrence patterns. However, remember that these OTUs vary widely in their abundances, and so similarly abundant OTUs will cluster together. Futhermore, because most of the OTUs in the dataset are in low abundance (rare), most of the heatmap is white, meaning that these OTUs have values close to zero.

####4.2 Ordination in R   
We will use the metaMDS function to apply non-metric multidimensional scaling analysis to ordinate our data. NMDS is a robust method that makes few assumptions about the data, so it is a good "default" choice for ordination in the absence of any other information. When plotting, we set `type = t` to show the sample IDs. 
```
braycurtis.mds <- metaMDS(braycurtis.d)

plot(braycurtis.mds, type="t")
```
We apply metaMDS to all of the resemblance matrices and compare them by dividing the plot space into 2 rows and 2 columns with the par() command. Inspect the plots- zoom in if you like. 
Questions to ponder:
_How are they different? How do unweighted metrics (Sørenson, unweighted UniFrac) differ in pattern from weighted metrics (Bray-Curtis, weighted UniFrac)?_
_How do phylogenetic metrics (weighted UniFrac, unweighted UniFrac) differ in pattern from taxonomic matrics (Bray-Curtis, Sørenson)?_ 
_What does this tell us about the relative importance of ecological characteristics of our communities in determining the overarching patterns we observe?_
```
sorenson.mds=metaMDS(sorenson.d)
weighted_u.mds=metaMDS(weighted_u.d)
unweighted_u.mds=metaMDS(unweighted_u.d)
par(mfrow=c(2,2))
plot(braycurtis.mds, type="t", main= "Bray-Curtis")
plot(sorenson.mds, type="t", main="Sorenson")
plot(weighted_u.mds, type="t", main="Weighted UniFrac")
plot(unweighted_u.mds, type="t", main="Unweighted UniFrac")
```

###5. Writing out (exporting) tables from R 
 
We will save our resemblance matrices to our working directory so that we don't have to re- calculate them every time we continue analyses, or if we want to use these matrices. Remember that resemblance matrices are actually very long vectors. R needs to think of these as tables so that it can write them out with the correct delimitations. Thus, we take the opposite approach as we did when we read in the table. We use the `as.matrix()` command to convert the vector to a table.
``` 
braycurtis=as.matrix(braycurtis.d)
```
Then, we use the `write.table()` command to export the data. In this command, the first argument is the object that we want to export, the second argument (given in quotations) is what we want the name of the exported file to be - don't forget the extension, we set row.names and col.names to TRUE because we have both, we specify that we want the exported file to be tab-delimited by setting the sep="\t" argument, and, finally, we specify quote=FALSE, as otherwise R will put every value in the exported table in quotes, which is annoying. Use the same commands with the Sorenson matrix. 
```
write.table(braycurtis, "BrayCurtis_even.txt",row.names=TRUE,col.names=TRUE,sep="\t", quote=FALSE)

sorenson=as.matrix(sorenson.d)

write.table(sorenson, "Sorenson_even.txt",row.names=TRUE,col.names=TRUE,sep="\t", quote=FALSE)
```
If you open these resemblance matrices in a spreadsheet program, you will notice another formatting issue. You must insert a top leftmost cell so that the sample IDs (column names) are shifted one over. This may seem confusing, but inspect one of the exported files in a spreadsheet and everything will be illuminated. 
###6. Hypothesis testing 
Welcome back! Open R studio and change directory to the analysis folder that you used yesterday and load the vegan package. Use the provided scripts as a guide and reminder as to how to perform the following tasks: 
1.	Re-read in the UniFrac resemblance matrices. Tell R that these are resemblance matrices using as.dist().  
2.	Read in the Bray-Curtis and Sørenson resemblance matrices. Tell R that these are resemblance matrices.  
We will go through the hypothesis tests with the Bray-Curtis resemblance, and then you can go then you will work through the remaining Sørenson, weighted UniFrac, and unweighted UniFrac on your own. Make a table of results and compare across tests and across resemblance metrics. 

####6.1 PERMANOVA (permuted analysis of variance)   
The vegan function `adonis()` performs permutated analysis of variance to test for differences (in centroid and/or dispersion) between treatment groups. The algorithm acts on the resemblance matrix, and links it to Treatment groups through the map file. We set it up exactly like the ANOVA, and the output is an ANOVA table, which we call a.table: 
```
ad=adonis(braycurtis.d~Treatment, data=map, permutations=999)

a.table=ad$aov.tab
```

Inspect a.table. _Is the global effect of Treatment significant?_

####6.2 Permuted multivariate analysis of beta-dispersion (PERMDISP)   
The vegan function `betadisper()` performs PERMDISP. This test is different from the others in that it specifically tests for differences in the spread (dispersion, variability) among groups.   
   
Therefore, if you use this test in combination with one of the other three, you will be able to tease apart whether groups of communities are different because they have different centroids or different spreads. For example, if PERMANOVA yields a significant difference, but PERMDISP does not, you can safely say that the distinction between groups can be attributed to differences in their centroid. Ordinations are often a good way to visually support and summarize these findings. 
```
b=betadisper(braycurtis.d, group=map[,"Treatment"], type="median") >b > b.perm=permutest(b, group=map[,"Treatment"], type="median", permutations=999, pairwise=TRUE)

b.perm 
```
Fill in the chart as you finish the tests:

|_Test_| Sorenson | Bray-Curtis | Weighted UniFrac | Unweighted UniFrac |
| :------------:|:-------------: | :-----: | :-----: | :-----: |
| PERMANOVA (adonis)|R2 =, p = | R2 =, p = | R2 =, p = | R2 =, p = |
| PERMDISP|Global p = , list significant pairwise differences | Global p = , list significant pairwise differences | Global p = , list significant pairwise differences | Global p = , list significant pairwise differences |

####6.3  How reproducible are replicates samples? 
In answering this question, we will also introduce how to use loops in R. Loops are a useful way to prevent yourself from doing tedious, repetitive calculations. Here, we write a loop to examine the reproducibility across replicates. Details for each step in the loop are provided in the R script. In the end, we plot the results to find that there is quite a bit of variability between some of the replicate samples.   
_Exercise: read in the full (not collapsed) tables), and then build a bray-curtis distance with them before starting the script_
```
fullmap=read.table(“FullMap.txt”)
otu.full=read.table(“otu_full.txt”)
 

u=unique(fullmap[,"Sample"])

meanreps.out=NULL

for(i in 1:length(u)){
  temp=braycurtis[map[,"Replicates"]==u[i],map[,"Replicates"]==u[i]]
  temp.d=as.dist(temp)
  m=mean(temp.d)
  meanreps.out=c(meanreps.out,m)
}

names(meanreps.out)=u
hist(meanreps.out)
meanreps.out
```

####6.4. Linking environmental gradients to community patterns 
####6.4.1. Making a resemblance matrix of time (or space). 
I have written a custom R function to create a time/space/environmental matrix to correlate to changes in the communities. It is called makeTimeDist.f. The .f signifies that this is a function and will require certain arguments to work. Inspect the syntax of the function in the provided R script, but don't worry too much about the mechanics of it now: 
```
makeTimeDist.f=function(map_file){
  map=map_file
  temp=as.matrix(map[,"Instar"])
 

  names(temp)=map[,"SampleID"]
  temp.d=dist(temp, method="manhattan", diag=FALSE)
  head(temp.d)
  temp.out=as.matrix(temp.d)
  colnames(temp.out)=map[,"SampleID"]
  row.names(temp.out)=map[,"SampleID"]
  print(head(temp.out))
  return(temp.d)
}
```
 
The important part is to be able to use the function. The argument for the function is map_file. Thus, you should provide your map file as input. We also need to name the output of the function, here we've named it time.d so that it is clear that it is a distance matrix. 
```
space.d=makeSpaceDist.f(map)
```
Inspect the head of this time matrix, provided automatically by the function. Do the values make sense with what you know about the instars for each sample? 
Now, we will use the non-parametric Mantel test `mantel()` to determine if changes in time correlate with community patterns.
``` 
mantel(time.d,braycurtis.d, method="pearson", permutations = 999)
```
How do we interpret the results from this test?  What is the test statistic?  What is the p-value?

We can perform the Mantel test using our other resemblances and compare: 
```
mantel(time.d,sorenson.d, method="pearson", permutations = 999)

mantel(time.d,weighted_u.d, method="pearson", permutations = 999)

mantel(time.d,unweighted_u.d, method="pearson", permutations = 999)
```

_ Based on the above tests, which resemblance metric is most informative for understanding space gradients? _
Time differences can also be tested using Mantel.  You can make a pair-wise difference in time matrix and substitute for the space matrix.

####6.4.2 Correspondence analysis 
We can also correlated changes in time with changes along an ordination axis of communities. This is sometimes desirable, as the most variability in communities is often explained by the first and second axes of ordination analyses. By focusing just on the variability along these two axes, we may be able to uncover clear patterns. 
Use the `cca()` and `envfit()` functions to link space/time data to the communities. 

```
otu.ca=cca(t(otu))

plot(otu.ca)

ev.instar=envfit(otu.ca, as.numeric(map[,"Instar"]), perm=1000)

ev.instar

plot(ev.instar)
```

Do these results support that there is a significant influence of time on community structure?

####6.4.3 Testing for time-treatment interactions. 
Now that we have evidence that there are differences between treatments (evidenced by PERMANOVA and PERMDISP global differences), and that there are correlations with time, we want to go back and use the `adonis()` function to perform a PERMANOVA to test for a time- treatment interaction. 
```
ad2=adonis(braycurtis.d~Treatment*as.numeric(Instar), data=map, permutations=999)

a.table2=ad2$aov.tab
a.table2
```

Do these results suggest a significant influence of space/time?

###6.5 Analyzing community patterns

As before, read in the Bray Curtis resemblance matrix, the OTU table, and the mapping file. Don't forget to assign the RDP column as "rdp" - we'll need these taxonomic IDs for today's analyses. 
```
braycurtis=read.table("BrayCurtis_even.txt", header=TRUE, row.names=1, sep="\t")
braycurtis.d=as.dist(braycurtis)
 otu=read.table("data_even.txt", header=TRUE, row.names=1,sep="\t", check.names=FALSE)

rdp=otu[,ncol(rdp)]

names(rdp)=row.names(otu)

otu=otu[,-(ncol(rdp))]

map=read.table("Manduca_map_sorted.txt", header=TRUE, sep="\t")
```

#####6.5.1 : Do different treatments/series have similar patterns?  PROTEST: Procrustean superimposition analysis 
We will use the `protest()` function from the vegan package to compare the time series of different treatments to one another. We want to know if the different treatments are synchronous (changing at the same general rate and direction, without requiring that the compositions are similar). For this to work, it is essential that the same observations are in the same order across treatments. PROTEST cannot read the sample IDs. 
First, designate treatment groups from the map file using the `unique()` function, we did previously. Then, use the custom-function `makeRedresem.f()`  (make reduced resemblance) to generate a resemblance matrix for each treatment. Inspect the function, and notice that the input is the full resemblance matrix (in table format), plus a Treatment ID (given in quotes), and that the output (return) is in vector format. The treatment ID must match one of those from the mapping file exactly. 
```
u=unique(map[,"Treatment"])

makeRedresem.f=function(resem_fp,group){  
resem=resem_fp
red=resem[map[,"Treatment"]==group, map[,"Treatment"]==group]
   red.d=as.dist(red)
   return(red.d)
} 

Trt1.d=makeRedresem.f(braycurtis,"TREATMENT1NAME")
Trt2.d=makeRedresem.f(braycurtis,"TREATMENT2NAME") …
```

In the end, we have as many "mini" resemblance matrix as treatments.  We sanity check to make sure they are in the same order by using “head” to make sure that each treatment had samples in consecutive order, then we use protest to ask if their overarching patterns are comparable.

```
protest(Trt1.d,Trt2.d, permutations=999)
```

Using PROTEST, are the two treatments correlated in their dynamics? 

If we use the mantel() function to test with the previously introduced Mantel test for linear matrix correlation, we get a similar result:
```
mantel(Trt1.d, Trt2.d, method="pearson", permutations=999)
```

Do both Mantel and PROTEST methods agree? 

#####6.5.2. Taxonomic Venn analysis:  The contributions of individual OTUs to community dynamics
We want to know which OTUs are shared among treatments, and which are unique to certain treatments. A taxonomic Venn analysis is the most straightforward and common method to do so. It is based on binary (presence/absence) data. First, we combine the instars for each treatment, and create smaller OTU table that we call "otu.trt", with each treatment as a column, and all OTU IDs in rows. For example, if any instar from Treatment 1 contained OTU 1, then there would be a "1" value in the Treatment 1 column in this OTU table. (You could do a similar analysis by instar, combining all treatments within each instar.). 
```
otu.trt=matrix(0,ncol=length(u),nrow=nrow(otu))

for(i in 1:length(u)){
  temp=otu[,map$Treatment==u[i]]
  dim(temp)
  temp2=rowSums(temp)
  otu.trt[,i]=temp2
}

row.names(otu.trt)=row.names(otu)
colnames(otu.trt)=u
```
Then, we install the library limma and load it. We use the function `vennCounts()` to calculate all OTUs that are unique and shared among all combinations of treaments, and then write out the results to the file we name "VennCounts.txt." 
> library(limma)
> v=vennCounts(otu.trt)
> #Write out the results of venncounts
> write.table(v, "VennCounts.txt", quote=FALSE, sep="\t")

Inspect the vennCounts output, which we called “v”. To interpret this output, the "1" says that the treatment group is included in the final count (rightmost column). This, there are 0 OTUs that are not included in any of the 6 groups, 63 OTUs that are unique to the MRS group, and 6 OTUs that are shared between the Lr and the MRS groups. 

It is often of interest to be able to identify exactly who those 63 unique OTUs were in the MRS treatment. We use the rdp names and the OTU IDs to create files of a list for each treatment combination. There are a total of 64 possible treatment combinations, but some of them have no OTUs. There will be one file written out for each combination that includes OTUs, but the label for each file must be referenced to the VennCounts.txt file to determine to which group the OTUs belong. Below is a custom script that I wrote to do the job, with VennOTUGroups as a vector of each OTU's assignment among the 64 possible Venn Treatment Group combinations: 
```
#Make a presence/absence (binary) table

otu.trt.pa=1*(otu.trt>0)
u=unique(map[,"Treatment"])

#This loop will output one text file per VennCounts #category that has a list of all ofthe OTU IDs and #their RDP taxonomic affiliations that belong to each #VennCount group.
#The group name is the row of the Venn Count, so you #will have to compare the VennCount file to the output #to determine which group.

tmp=NULL
VennOTUGroups=vector(length=nrow(otu)
for(y in 1:nrow(v)){
  if(v[y,ncol(v)]!=0){
    for(z in 1:nrow(otu.trt.pa)){
      if(sum(1*(as.vector(otu.trt.pa[z,]) == as.vector(v[y,(1:ncol(v)-
1)])))==length(u)){
        VennOTUGroups[z]=y
        n=c(row.names(otu.trt.pa)[z],paste(rdp[z]))
        tmp=rbind(tmp,n)
} } 
    write.table(tmp, paste("VennCounts_Condition_",y,"_OTUList.txt",sep=""),
row.names=FALSE, sep="\t", quote=FALSE)
    print(dim(tmp))
  }
tmp=NULL }
```
We check the output to make sure it makes sense with what we expect (and that the script is working correctly). First, we check the length of VennOTUGroups - this should be the same length as the number of OTUs that we have (389). Then, we use a logical statement, "VennOTUGroups==2" to sum the number of times that it is TRUE that VennOTUGroups has a 2. From the head of v, we know that there should be 63 OTUs that are only viewed in MRS, which is VennOTUGroups 2. We could check other groups as well. There are 16 OTUs shared across all groups (Venn Group 64), which also matches v. 
```
length(VennOTUGroups)
sum(1*(VennOTUGroups==2))
sum(1*(VennOTUGroups==64))
```
#####6.5.3 Clustering OTUs by similar dynamics 
By making use of the abundance data of OTUs, we can also identify OTUs that have common dynamics. This especially is useful when considering temporal or spatial series. First, we make an OTU table standardized by rows (OTU patterns) so that prevalent and rare taxa that have similar dynamics will be clustered together. Otherwise, the abundances of OTUs will drive the patterns, and not their dynamics. Then, we use the `hclust()` function (hierarchical clustering) based on Bray-Curtis dissimilarities calculated between all pairs of OTUs (rather than all pairs of samples/communities, as we did previously). The resulting dendrogram is large (as many nodes as OTUs), but informative. 
```
rSums=rowSums(otu)
otu.relrows=otu/rSums
braycurtis.rows.d=vegdist(otu.relrows, method="bray")
otu.cluster=hclust(braycurtis.rows.d, method="complete")
plot(otu.cluster)
```
We can also label nodes by Venn Groups, or any other categorical value, like rdp names (abbreviated using the `make.cepnames()` function from vegan) 
```
plot(otu.cluster,labels=VennOTUGroups)
rdp2=make.cepnames(rdp)
plot(otu.cluster, labels=rdp2)
```
Explore these plots in detail on your own time, and also explore the related functions to hclust given at the bottom of the R help files, under "See Also". 

#####6.5.4 Species abundance and species occurrence distributions 
The vegan package has a few options for exploring and plotting species abundance distributions. The most common is called using the `radfit()` function (aka rank abundance distribution). We apply this function to rSums, a vector of rowSums (OTU occurrences) calculated from the OTU table. The algorithm fits a few different but commonly used SAD models so that you can compare them. 
```
r.sad=radfit(rSums)
r.sad
```
The output from radfit provides some metrics for evaluating the fit of the model, as well as a built-in plotting algorithm. Here we can use either the Akaike (AIC) or the Bayesian (BIC) Information Criterion to evaluate the fit of the model- in both cases, a lower number is a better fit. In actuality, none of these models are awesome fits, but Mandelbrot is best. We can also see from the plot that the Mandelbrot function fits best. 
```
plot(r.sad, las=1,main = "Manduca species abundance distribution", ylab="Abundance(No. sequences)", xlab="OTUs ranked by abundance")
```
We use the same strategy for species occurrence distributions, but as input we use the total number of occurrences per OTU (out of 76 total observations) instead of the total OTU abundances. From this, there are a few models that are equally acceptable: 
```
otu.pa=1*(otu>0)
rSums.pa=rowSums(otu.pa)
r.sod=radfit(rSums.pa)
r.sod

plot(r.sod, las=1,ylab="Occurrence (out of 76 total observations)", xlab="OTUs ranked by occurrence", main="species occurrence distribution")
```
#####6.5.5 MultiCOLA 
I've written a custom script to perform MultiCOLA at different species cut-offs, as per Gobet et al. 2010. The authors of the work also provide some R scripts that you could use instead. I used Mantel tests to compare the "full" dataset with each "reduced" dataset, but you could also use PROTEST. The output of the custom function is a table of cut-offs, number of remaining OTUs, and the Mantel correlation R and p-value for significance.
``` 
MantelMultiCOLA.f=function(otu_fp){
  #Step 1.  Read in full dataset:
  otu2=otu_fp
  library(vegan)
  cutoff=c(1.00,0.90, 0.80, 0.70, 0.60, 0.50, 0.40, 0.30, 0.20, 0.10, 0.05, 0.025,
0.01, 0.005, 0.001)
  m.out=NULL
  otu.pa=1*(otu2>0)
  r=rowSums(otu2)
  #Create a vector of indexes of the ranked OTUs
  r2=sort(r,decreasing=TRUE, index=TRUE)
  r2$ix
  r2$x
  for(j in 1:length(cutoff)){
    print(cutoff[j])
    no.keep=ceiling(cutoff[j]*nrow(otu2))
    otu.index.keep=r2$ix[1:no.keep]
    otu.keep=otu2[otu.index.keep,]
    print(head(otu.keep))
#Write out otu tables at each cutoff
    write.table(otu.keep, paste(u[i],"_",cutoff[j],"_otu.txt", sep=""),sep="\t",
quote=FALSE)
    all.dist=vegdist(t(otu2), method="bray")
    subset.dist=vegdist(t(otu.keep),method="bray")
    m1=mantel(all.dist,subset.dist, method="pearson",permutations=999)
    m=c(paste(cutoff[j]),m1$statistic,m1$signif,dim(otu.keep)[1])
    m.out=rbind(m.out,m)
  }
  colnames(m.out)=c("Cutoff", "AllvSubsetPearsonR", "AllvSubset_pvalue",
"NoOTUsSubset")
  #write.table(m.out, "MantelMultiCOLA.txt", sep="\t", quote=FALSE, row.names=FALSE)
  return(m.out)
}
```

Run the function, and name the output "MultiCOLA.test" 
```
MultiCOLA.test=MantelMultiCOLA.f(otu)
```
Inspect the output, MultiCOLA.test. The first cut-off is not a cut-off at all, as it contains 100% of the most prevalent OTUs. Where do the patterns start to break down? 
```
MultiCOLA.test
```


