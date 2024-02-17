# UNIX_ass_chaves_S24
Unix assignment for EEOB 546 Spring 2024
## Manuela Chaves  

## Part 1: Data Inspection 

During this UNIX assignment, two raw genomic data files (“fang _et_al_genotypes.txt” and “snp_position.txt”) were subjected to a file inspection process through UNIX commands, to describe and understand the kind of data we are dealing with.   

### Attributes of the Fang et al & SNP files 

First, I wanted to open the files and I tried the cat command, followed by the head command to view the categories (headers) of the data. 

``` 

cat “fang_et_al_genotypes.txt” 

cat “snp_position.txt” 

Head –n 4 “fang_et_al_genotypes.txt” 

Head –n 4 “snp_position.txt” 

``` 

However, I realized this visualization was not very user-friendly for my purposes. Then, I decided to use the less command instead. 

``` 

less “fang_et_al_genotypes.txt” 

less “snp_position.txt” 

``` 

This time I was able to observe more organized data with rows and columns, with several genomic categories, including SNP ID, genotype group and chromosome positions.  

To make a more quantitative analysis of the data, I run the following commands: 

``` 

wc fang_et_al_genotypes.txt 

awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt 

wc snp_position.txt” 

awk -F "\t" '{print NF; exit}' snp_position.txt 

Du –h “fang_et_al_genotypes” 

Du –h “snp_position.txt” 

```  

I learnt that the fang et al file and the SNP file were composed of, respectively: 

Fang et al: 986 columns - 2783 lines - 2744038 words - 11051939 characters 

SNP: 15 columns - 984 lines – 13198 words - 82763 characters 

This description assumes that the SNP file is smaller than the Fang et al. And it is, since its size is 49K compared to 6.7M. However, the SNP file contains complementary information among its 15 columns, and the main difference between the two data files is that the SNP ID is organized in columns in Fang et al, while in the SNP file it is organized in rows.    

 

## Part 2: Data Processing 

Before editing data, I copied it to keep the integrity of the raw data. I also created subfolders for the maize and teosinte genotypes to make data processing more manageable. The files created through the assignment for maize and teosinte were stored in their corresponding folders.  

``` 

Mkdir maize  

Mkdir teosinte  

cp <name_files> <directory/maize> 

cp <name_files> <directory/teosinte> 

```  

### Extracting data  

In order to obtain the different files for the assignment, it is necessary to go through several steps: 

1. Data transposing
2. Data sorting
3. Data extraction 

#### 1. Data transposing: 

For the assignment we need to extract the Maize groups (ZMMIL, ZMMLR, ZMMMR) and the Teosinte group (ZMPBA, ZMPIL, ZMPJA) separately. These groups then need to be organized in a file “such that the first column is "SNP_ID", the second column is "Chromosome", the third column is "Position", and subsequent columns are genotype data from either maize or teosinte individuals” 

To do this, first I selected the headers and grep a specific part of the group name for each group (ZMM for maize and ZMP for Teosinte) and extracted all the columns related in the Fang file. Then, I redirected the information into a new file named “maize_phenotype” and “teosinte_phenotype” respectively.  

```  

(head -n 1 fang_et_al_genotypes.txt && grep 'ZMM' fang_et_al_genotypes.txt) | cut -f 4-986 > maize_phenotype.txt 

(head -n 1 fang_et_al_genotypes.txt && grep 'ZMP' fang_et_al_genotypes.txt) | cut -f 4-986 > teosinte_phenotype.txt 

``` 

However, we need the SNP IDs to be organized in rows, not columns, to extract and merge the information we need. Thus, I used the awk command to transpose the data: 

```  

awk -f transpose.awk maize_phenotype.txt > transposed_maize_phenotype.txt  

awk -f transpose.awk teosinte_phenotype.txt > transposed_teosinte_phenotype.txt 

``` 

#### 2. Data sorting: 

Once I got both files organized in the same manner, I sorted the data by SNP ID (-k1,1). The file was stored in maize/teosinte_sorted.txt 

```  

sort -k1,1 transposed_maize_phenotypes.txt > maize_sorted.txt 

sort -k1,1 transposed_teosinte_phenotype.txt > teosinte_sorted.txt  

``` 

#### 3. Data extraction: 

The Fang et al file is already ready to get the information we need. However, we still need to process the SNP file: by cutting the columns SNP ID, Chromosome and Position respectively, then sorting and storing the data without the headers.  

```  

cut -f 1,3,4 snp_position.txt | sed '1d' | sort -k1,1 > snp_position_sorted.txt  

``` 

Then, both sorted data from the Fang and SNP files were merged into one, making use of the tab delimitation option, and selecting the first column from both files (-1 1 -2 1 , represent the SNP ID). 

 ``` 

join -t $'\t' -1 1 -2 1 snp_position_sorted.txt maize_sorted.txt > maize_snp_sorted.txt 

join -t $'\t' -1 1 -2 1 snp_position_sorted.txt teosinte_sorted.txt > teosinte_snp_sorted.txt 

```  

To confirm that all columns were merged, I used awk to count the number of columns in the individual files and compared it with the final column number. As we have one column with the same data in both files (SNP ID), it merged into one single column. The result is a file ordered by SNP ID, Chromosome and position. 

#### Extracting data of SNP ID: Unknown and Multiple  

Using the awk command, I selected the respective columns needed from the merged file, specifying the unknown or multiple groups. 

```  

awk -v OFS='\t' 'BEGIN {print "SNP-ID", "Chromosome", "Location", "Genotype_Data"} /unknown/ {print}' maize_snp_sorted.txt > unknown_maize_SNP.txt 

awk -v OFS='\t' 'BEGIN {print "SNP-ID", "Chromosome", "Location", "Genotype_Data"} /multiple/ {print}' maize_snp_sorted.txt > multiple_maize_SNP.txt 

``` 

### Ordering data 

According to the assignment, we need to order the sorted merged file so that the SNP ID: (l) increases, and the missing information is represented as ‘?’ and (ll) decreases, and the missing information is represented as ‘-’.  

For this, I used a for loop command, stating that “for chromosomes from 1 to 10, Sort data by position (column 3), and enumerate each new file by its specific chromosome (i)”. To state the decreasing sorting, I used the r option for the sort. 

The missing data is represented as ‘?’ by default. Then, for the ‘-’ representation, I integrated a sed command to substitute the ‘?’ For ‘–’ in a global manner, into the for loop for the decreasing files. 

``` 

for ((i=1; i<=10; i++)); do awk -v i="$i" '{if ($2==i) print $0}' maize_snp_sorted.txt | sort -k3,3n | awk -v OFS='\t' 'BEGIN {print "SNP_ID", "Chromosome", "Position", "Genotype_data"}{print}'> chr"$i"_maize_increasing.txt; done 

for ((i=1; i<=10; i++)); do awk -v i="$i" '{if ($2==i) print $0}' maize_snp_sorted.txt | sort -k3,3nr | sed 's/?/-/g' | awk -v OFS='\t' 'BEGIN{print "SNP_ID", "Chromosome", "Position", "Genotype_data"}{print}'> chr"$i"_maize_decreasing.txt; done 

``` 

## Data organization and uploading to GitHub 

Finally, I created new folders and moved the data into their respective folders. I also kept the raw data separately:  

- Maize/teosinte_multiple 
- Maize/teosinte_unknown
- Maize/teosinte_decreasing
- Maize/teosinte_increasing
- Raw_data   

All data can be found in the GitHub repository https://github.com/chavesMM/UNIX_ass_chaves_S24.git 
