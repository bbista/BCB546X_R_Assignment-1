---
output:
  html_document: default
  pdf_document: default
---
# **BCB546X_R_Assignment**
## Data Import
Used the readr "read_tsv" function (tab delimited .txt files) to read the files from BCB546X Git repository using URL for the raw file.

```{r}
library(tidyverse)
Fang <- read_tsv("https://raw.githubusercontent.com/EEOB-BioData/BCB546X-Fall2018/master/assignments/UNIX_Assignment/fang_et_al_genotypes.txt")
SNP_Position <- read_tsv("https://raw.githubusercontent.com/EEOB-BioData/BCB546X-Fall2018/master/assignments/UNIX_Assignment/snp_position.txt")
```

## Data Inspection
Inspected type, size and length of the data frame from the grid view of RStudio environment panel. Also used the ncol(), nrow(), and dim() functions with data frames.

```{r}
ncol(Fang)
nrow(Fang)
dim(Fang)
str(Fang)
ncol(SNP_Position)
nrow(SNP_Position)
dim(SNP_Position)
str(SNP_Position)
```

## Data Processing
Subset data for Maize and Teosinte groups from the Fang genotypes dataframe using the which() function and logical operator "==". Separated the required information from SNP position dataframe. Transposed the Maize and Teosinte genotype data subsets using t(), assigned column names and removed extra rows. 

```{r}
### Maize and Teosinte subsets
Fang_maize <- Fang[which(Fang$Group=="ZMMIL" | Fang$Group =="ZMMLR" | Fang$Group == "ZMMMR"),]
Fang_teosinte <-Fang[which(Fang$Group=="ZMPBA" | Fang$Group =="ZMPIL" | Fang$Group == "ZMPJA"),]
#### Inspected the files
head(Fang_maize)
head(Fang_teosinte)
### Separated required columns from the SNP position data
SNP_reduced <- SNP_Position[,c(1,3,4)]
### Transposed genotype subsets, used Sample ID as column name and removed extra rows
Maize_transposed <- as.data.frame(t(Fang_maize))
colnames(Maize_transposed) <- as.character(unlist(Maize_transposed[1,]))
Maize_reduced <- Maize_transposed[-c(1:3),]
Teosinte_transposed <- as.data.frame(t(Fang_teosinte))
colnames(Teosinte_transposed) <- as.character(unlist(Teosinte_transposed[1,]))
Teosinte_reduced <- Teosinte_transposed[-c(1:3),]
```

Merged the reduced SNP position and Fang maize and teosinte genotype dataframes. 

```{r}
Maize_merged <- merge(x = SNP_reduced, y = Maize_reduced, by.x = "SNP_ID", by.y ="row.names", all.y = TRUE)
Teosinte_merged <- merge(x = SNP_reduced, y = Teosinte_reduced, by.x = "SNP_ID", by.y ="row.names", all.y = TRUE)
### Checked merge using colnames()
```

Edited the dataframes to get SNP position in increasing value, split the merged data for each of the 10 chromosomes and generated corresponding files using write.csv

```{r}
Maize_merged <- Maize_merged[order(Maize_merged$Position),]
Teosinte_merged <- Teosinte_merged[order(Teosinte_merged$Position),]
### Writing files
for (a in 1:10){
  maize_ordered_chromosome <- Maize_merged[Maize_merged$Chromosome == a, ]
  write.csv(maize_ordered_chromosome, file= paste("Maize_chromosome", a, ".csv", sep=""), row.names = F)
}
for (a in 1:10){
  teosinte_ordered_chromosome <- Teosinte_merged[Teosinte_merged$Chromosome == a, ]
  write.csv(teosinte_ordered_chromosome, file= paste("Teosinte_chromosome", a, ".csv", sep=""), row.names = F)
}
```

###### Filename: group_chromosome.csv
Edited the dataframes to get SNP position in decreasing value, split the merged data for each of the 10 chromosomes, replaced '?' with '-' for missing data and generated corresponding files using write.csv

```{r}
Maize_merged_r <- Maize_merged[order(Maize_merged$Position, decreasing=T),]
Teosinte_merged_r <- Teosinte_merged[order(Teosinte_merged$Position, decreasing=T),]
### Confirmed decreasing position order using the View() function
### Replacing symbol for missing data and writing files
for (a in 1:10){
  maize_reverse_chromosome <- Maize_merged_r[Maize_merged_r$Chromosome == a, ]
  write.csv(maize_reverse_chromosome, file= paste("Maize_chromosome", a, "r.csv", sep=""), row.names = F)
}
for (a in 1:10){
  teosinte_reverse_chromosome <- Teosinte_merged_r[Teosinte_merged_r$Chromosome == a, ]
  write.csv(teosinte_reverse_chromosome, file= paste("Teosinte_chromosome", a, "r.csv", sep=""), row.names = F)
}
```

###### Filename: group_chromosomer.csv
## Data Visualization
Loaded packages

```{r}
library(ggplot2)
library(tidyverse)
library(dplyr)
library(reshape2)
library(plyr)
```

Transposed the Fang genotype dataset and merged with reduced SNP position dataset (ID, Chromosome and Position)

```{r}
Fang_transposed <- as.data.frame(t(Fang))
colnames(Fang_transposed) <- as.character(unlist(Fang_transposed[1,]))
Fang_SNP_merged <- merge(x = SNP_reduced, y = Fang_transposed, by.x = "SNP_ID", by.y ="row.names", all.y = TRUE)
```

#### SNPs per chromosome
Used ggplot() and the merged dataframe.
```{r}
ggplot(Fang_SNP_merged, aes((Chromosome))) + geom_bar() + ggtitle("Number of SNPs for Each Chromosome") + 
  labs(x="Chromosome",y="SNP Count")
```

#### SNPs in different groups
Used ggplot() and the Fang dataframe.
```{r}
ggplot(Fang, aes((Group))) + geom_bar() + labs(x="Group",y="SNP Count")
```

The Maize groups ZMMLR and ZMMIL as well as the Teosinte group ZMPIL contribute most of the SNPs. 

#### Missing data and heterozygosity
Reshaped Fang genotypes dataframe.
```{r}
headers <- colnames(Fang)[-c(1:3)]
Fang_melt <- melt(Fang, measure.vars = headers)
### Missing data > NA
Fang_melt[ Fang_melt == "?/?" ] = NA
### Column for homozygous genotypes
Fang_melt$isHomozygous <- (Fang_melt$value=="A/A" | Fang_melt$value=="C/C" | Fang_melt$value=="G/G" | Fang_melt$value=="T/T")
### Fang genotypes ordered by Sample ID 
Fang_ID <- Fang_melt[order(Fang_melt$Sample_ID),]
### Proportion of heterozygous and homozygous sites
ID_count <- ddply(Fang_ID, c("Sample_ID"), summarise, counting_homozygous=sum(isHomozygous, na.rm=TRUE), counting_heterozygous=sum(!isHomozygous, na.rm=TRUE), isNA=sum(is.na(isHomozygous)))
ID_count_melt <- melt(ID_count, measure.vars = c("counting_homozygous", "counting_heterozygous", "isNA"))
### Plot using ggplot()
ggplot(ID_count_melt,aes(x = Sample_ID, y= value, fill=variable)) + geom_bar(stat = "identity", position = "stack")
### Fang genotypes ordered by group
Fang_group <- Fang_melt[order(Fang_melt$Group),]
### Proportion of heterozygous and homozygous sites
Group_count <- ddply(Fang_group, c("Group"), summarise, counting_homozygous=sum(isHomozygous, na.rm=TRUE), counting_heterozygous=sum(!isHomozygous, na.rm=TRUE), isNA=sum(is.na(isHomozygous)))
Group_count_melt <- melt(Group_count, measure.vars = c("counting_homozygous", "counting_heterozygous", "isNA"))
### Plot using ggplot()
ggplot(Group_count_melt,aes(x = Group, y= value, fill=variable)) + geom_bar(stat = "identity", position = "stack")
```

#### Heterozygosity according to SNP
Visualized heterozygosity according to SNP ID usingthe same code as above with merged Fang genotype and SNP position dataframes. 
```{r}
headers_m <- colnames(Fang_SNP_merged)[-c(1:3)]
merged_m[ merged_m == "?/?" ] = NA
merged_m$isHomozygous <- (merged_m$value=="A/A" | merged_m$value=="C/C" | merged_m$value=="G/G" | merged_m$value=="T/T")
merged_m_SNP <- merged_m[order(merged_m$SNP),]
SNP_hetero <- ddply(merged_m_SNP, c("SNP_ID"), summarise, heterozygocity_count=sum(!isHomozygous, na.rm=TRUE), total_count=sum(!is.na(isHomozygous)))
SNP_hetero$Heterozygocity <- (SNP_hetero$heterozygocity_count/SNP_hetero$total_count)
SNP_hetero_melt <- melt(SNP_hetero, measure.vars = "Heterozygocity")
ggplot(SNP_hetero_melt,aes(x = SNP_ID, y= value, fill=variable)) + geom_bar(stat = "identity", position = "stack")
```