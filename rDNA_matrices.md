#scripts modified from Dovetail Genomics MicroC analysis pipeline ( 

#uploaded a new genome file called rdna.fa that only includes the rdna sequence from the right side 244000-end
#creating indexed genome files compatible with bwa and pairix
samtools faidx rdna.fa
cut -f1,2 rdna.fa.fai > rdna.genome
bwa index rdna.fa

#bwa and samtools for analysis and alignment
mkdir tmpfiles

bwa mem -5SP -T0 -t16 rdna.fa <raw data R1 filepath> <raw data R2 filepath> | pairtools parse --min-mapq 0 --walks-policy 5unique --max-inter-align-gap 30 --nproc-in 8 --nproc-out 8 --chroms-path <path to indexed genome file> | pairtools sort --tmpdir=tmpfiles/ --nproc 16|pairtools dedup --nproc-in 8 --nproc-out 8 --mark-dups --output-stats <stats file name>.txt | pairtools split --nproc-in 8 --nproc-out 8 --output-pairs <pairs file name>.pairs --output-sam -|samtools view -bS -@16 | samtools sort -@16 -o <output file name>.PT.bam; samtools index <file name>.PT.bam


#to print the stats
python3 Micro-C/get_qc.py -p <stats file name>.txt

#running pairix to index the samples for .cool matrix
#first zip the files and then run pairix
bgzip <pairs file name>.pairs

~/MicroC/Spombe/Mar26/pairix/bin/pairix <pairs file name>.pairs.gz

#mapping to the genome file, bin size 100bp
cooler-0.8.9 cload pairix -p 8 rdna.genome:100 <pairs file name>.pairs.gz <cool file name>.cool

#balancing the file 
cooler-0.8.9  balance -p 8 <cool file name>.cool
