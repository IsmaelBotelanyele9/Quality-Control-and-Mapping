# Quality-Control-and-Mapping

#!/bin/bash

# Script for Quality Control and alignment of ERR6484153_1 & ERR6484153_2 and alignment with GCA_000005845.2_ASM584v2 & GCF_000005845.2_ASM584v2
# For demonstration or training purpose only


# Create directories
mkdir Fasta Fastq QCFiles AligFiles


# Download data/Fastq files
wget -P ~/home/ismael/Ecoli/Fastq/ ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR490/SRR490124/SRR490124_1.fastq.gz
wget -P ~/home/ismael/Ecoli/Fastq/ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR490/SRR490124/SRR490124_2.fastq.gz


echo "Run Prep Files..."


############################################################# Prep Files (TO BE GENERATED ONLY ONCE) #################################################################


# Download Reference Genome
wget -P ~/home/ismael/Ecoli/Fasta/ https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_000005845.2
wget -P ~/home/ismael/Ecoli/Fasta/ https://www.ncbi.nlm.nih.gov/datasets/genome/GCA_000005845.2

mv ~/home/ismael/Ecoli/Fasta/ GCF_000005845 GCF_000005845.2_ASM584v2_genomic.fna
mv ~/home/ismael/Ecoli/Fasta/ GCA_000005845 GCA_000005845.2_ASM584v2_genomic.fna

############################################################# QC & Mapping Steps ######################################################################################


# Define directories
Fasta="/home/ismael/Ecoli/Fasta/GCF_000005845.2_ASM584v2_genomic.fna"
Fastq="/home/ismael/Ecoli/Fastq"
QCFiles="/home/ismael/Ecoli/QCFiles"
AligFiles="/home/ismael/Ecoli/AligFiles"

#-------------------------------
# STEP 1: Index the reference using BWA-INDEX
#-------------------------------

echo "STEP 1: Index the reference using BWA-INDEX"

bwa index ${Fasta}

# --------------------------------
# STEP 2: QC - Run Fastp
#---------------------------------

echo "STEP 2: QC - Run Fastp"

fastp -i ${Fastq}/ERR6484153_1.fastq.gz -I ${Fastq}/ERR6484153_2.fastq.gz -o ${QCFiles}/ERR6484153_1.fastp.gz -O ${QCFiles}/ERR6484153_2.fastp.gz

#---------------------------------
# STEP 3: Map to the reference using BWA-MEM
#---------------------------------

echo "STEP 3: Map to the reference using BWA-MEM"

bwa mem ${Fasta} ${QCFiles}/ERR6484153_1.fastp.gz ${QCFiles}/ERR6484153_2.fastp.gz > ${AligFiles}/ERR6484153.paired.sam

#---------------------------------
# STEP 4: Convert sam into bam using SAMTOOLS-VIEW
#---------------------------------

echo "STEP 4: Convert sam into bam using SAMTOOLS-VIEW"

samtools  view -b ${AligFiles}/ERR6484153.paired.sam > ${AligFiles}/ERR6484153.paired.bam

#---------------------------------
# STEP 5: Sort the bam files using SAMTOOLS-SORT
#---------------------------------

echo "STEP 5: Sort the bam files using SAMTOOLS-SORT"

samtools sort ${AligFiles}/ERR6484153.paired.bam > ${AligFiles}/ERR6484153.paired.sorted