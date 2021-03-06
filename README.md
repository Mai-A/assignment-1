# assignment
mkdir ~/workdir/assignment
cd ~/workdir/assignment
wget -c ftp://ftp-trace.ncbi.nih.gov/sra/sra-instant/reads/ByRun/sra/SRR/SRR879/SRR8797509/SRR8797509.sra
mv DRR000033.sra sample.sra
sudo apt install sra-toolkit
# convert sra to one file fastq
fastq-dump --split-spot sample.sra  
# get 5 samples each one of million reads
gzip sample.fastq
seqkit split sample.fastq.gz -s 1000000
# shuffle
seqkit shuffle sample.fastq.gz > shuffled.fastq.gz
# get 5 shuffled samples each one of million reads
seqkit split shuffled.fastq.gz -s 1000000
# use FASTQC to report the difference between S1_1 and S1_2
cd shuffled.fastq.gz.split
fastqc  --noexsample.fastq.gz.splittract -f fastq shuffled.part_001.fastq.gz
cd ..
cd sample.fastq.gz.split
fastqc  --noexsample.fastq.gz.splittract -f fastq sample.part_001.fastq.gz
# Mild Trimming for SX_1. {unshuffled}
sudo apt install cutadapt
cutadapt -m 10 -q 20 -o sample1_1_trimmed.fastq.gz sample.part_001.fastq.gz
# Aggressive Trimming for SX_2. {shuffled}
cd ..
cd shuffled.fastq.gz.split
cutadapt -m 50 -q 100 -o sample1_2_trimmed.fastq.gz shuffled.part_001.fastq.gz
# BWA Alignment
cd ..
source activate ngs1
mkdir -p ~/workdir/bwa_align/bwaIndex
cd ~/workdir/bwa_align/bwaIndex
ln -s ~/home/ngs-01/workdir/assignment/chr22_with_ERCC92.fa.
bwa index -a bwtsw chr22_with_ERCC92.fa.bwa 
cd ~/workdir/bwa_align
R1="$HOME/workdir/assignment/sample.fastq.gz.split/sample.part_001.fastq.gz"
R2="$HOME/workdir/assignment/sample.fastq.gz.split/sample.part_002.fastq.gz"
R3="$HOME/workdir/assignment/sample.fastq.gz.split/sample.part_003.fastq.gz"
R4="$HOME/workdir/assignment/sample.fastq.gz.split/sample.part_004.fastq.gz"
R5="$HOME/workdir/assignment/sample.fastq.gz.split/sample.part_005.fastq.gz"
# hisat2 Alignment
source activate ngs1
conda install -c bioconda hisat2 
mkdir -p ~/workdir/hisat_align/hisatIndex
cd ~/workdir/hisat_align/hisatIndex
hisat2_extract_splice_sites.py ~/workdir/assignment/chr22_with_ERCC92.gtf > splicesites.tsvn -s ~/home/ngs-01/workdir/assignment/chr22_with_ERCC92.fa.
hisat2_extract_exons.py ~/workdir/assignment/chr22_with_ERCC92.gtf > exons.tsv
hisat2-build -p 1 --ss splicesites.tsv --exon exons.tsv chr22_with_ERCC92.fa chr22_with_ERCC92
cd ~/workdir/hisat_align
R1="$HOME/workdir/assignment/shuffled.fastq.gz.split/HBR_Rep1_ERCC-Mix2_Build37-ErccTranscripts-chr22.read1.fastq.gz"
# Prepare the SAM file for assembly
source activate ngs1
conda install samtools
# Assembly with stringtie
source activate ngs1
conda install stringtie




