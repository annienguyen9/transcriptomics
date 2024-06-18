# **Adding Marker Gene to an Existing Reference Genome:**

There are cases where the publicly available GTF and FASTA files will not contain information for some of the genes expressed in a given sample. 
A transgenic sample is a good example of when you would not expect a gene of interest to be in the reference. Here, I am adding the Ai9 tdTomato sequence
to the reference mouse genome, following the 10x tutorial: https://www.10xgenomics.com/support/software/cell-ranger/latest/tutorials/cr-tutorial-mr

### 1. Download the reference genome you intend to add your transgenic sequence to.

Here I am using the 10x mouse reference genome.
![image](https://github.com/annienguyen9/transcriptomics/assets/167817503/26e41f30-dc82-4821-8898-752453a8f747)
Note: you may have to make an account to download any resources.

In Sherlock, use the command:
```
wget "https://cf.10xgenomics.com/supp/cell-exp/refdata-gex-GRCm39-2024-A.tar.gz"
```

### 2. Unzip the downloaded reference genome:
```
gunzip refdata-gex-GRCm39-2024-A.tar.gz
```
Note: any file ending in .gz can be unzipped with the 'gunzip' command.

*Optional: Delete the download file to save space:
```
rm refdata-gex-GRCm39-2024-A.tar.gz
```

### 3. Navigate to the FASTA folder in the opened reference genome file

### 4. Generate a new FASTA file for your transgenic sequence
4a. Acquire the Ai9 tdTomato sequence.
Andrea used https://www-ncbi-nlm-nih-gov.laneproxy.stanford.edu/nuccore/55420622 (GenBank: AY678269.1), but note that Ai75 signal will be the future nuclear reporter

FASTA INFO:
>AY678269.1 Synthetic construct tandem-dimer red fluorescent protein gene, complete cds
ATGGTGAGCAAGGGCGAGGAGGTCATCAAAGAGTTCATGCGCTTCAAGGTGCGCATGGAGGGCTCCATGA
ACGGCCACGAGTTCGAGATCGAGGGCGAGGGCGAGGGCCGCCCCTACGAGGGCACCCAGACCGCCAAGCT
GAAGGTGACCAAGGGCGGCCCCCTGCCCTTCGCCTGGGACATCCTGTCCCCCCAGTTCATGTACGGCTCC
AAGGCGTACGTGAAGCACCCCGCCGACATCCCCGATTACAAGAAGCTGTCCTTCCCCGAGGGCTTCAAGT
GGGAGCGCGTGATGAACTTCGAGGACGGCGGTCTGGTGACCGTGACCCAGGACTCCTCCCTGCAGGACGG
CACGCTGATCTACAAGGTGAAGATGCGCGGCACCAACTTCCCCCCCGACGGCCCCGTAATGCAGAAGAAG
ACCATGGGCTGGGAGGCCTCCACCGAGCGCCTGTACCCCCGCGACGGCGTGCTGAAGGGCGAGATCCACC
AGGCCCTGAAGCTGAAGGACGGCGGCCACTACCTGGTGGAGTTCAAGACCATCTACATGGCCAAGAAGCC
CGTGCAACTGCCCGGCTACTACTACGTGGACACCAAGCTGGACATCACCTCCCACAACGAGGACTACACC
ATCGTGGAACAGTACGAGCGCTCCGAGGGCCGCCACCACCTGTTCCTGGGGCATGGCACCGGCAGCACCG
GCAGCGGCAGCTCCGGCACCGCCTCCTCCGAGGACAACAACATGGCCGTCATCAAAGAGTTCATGCGCTT
CAAGGTGCGCATGGAGGGCTCCATGAACGGCCACGAGTTCGAGATCGAGGGCGAGGGCGAGGGCCGCCCC
TACGAGGGCACCCAGACCGCCAAGCTGAAGGTGACCAAGGGCGGCCCCCTGCCCTTCGCCTGGGACATCC
TGTCCCCCCAGTTCATGTACGGCTCCAAGGCGTACGTGAAGCACCCCGCCGACATCCCCGATTACAAGAA
GCTGTCCTTCCCCGAGGGCTTCAAGTGGGAGCGCGTGATGAACTTCGAGGACGGCGGTCTGGTGACCGTG
ACCCAGGACTCCTCCCTGCAGGACGGCACGCTGATCTACAAGGTGAAGATGCGCGGCACCAACTTCCCCC
CCGACGGCCCCGTAATGCAGAAGAAGACCATGGGCTGGGAGGCCTCCACCGAGCGCCTGTACCCCCGCGA
CGGCGTGCTGAAGGGCGAGATCCACCAGGCCCTGAAGCTGAAGGACGGCGGCCACTACCTGGTGGAGTTC
AAGACCATCTACATGGCCAAGAAGCCCGTGCAACTGCCCGGCTACTACTACGTGGACACCAAGCTGGACA
TCACCTCCCACAACGAGGACTACACCATCGTGGAACAGTACGAGCGCTCCGAGGGCCGCCACCACCTGTT
CCTGTACGGCATGGACGAGCTGTACAAGTAA

4b. Paste this sequence into a new nano text editor file
```
nano Ai9tdT_orig.fa
```
4c. Save this sequence as an original FASTA file:
```
Ai9tdT_orig.fa
```

### 5. Edit this file so that the header only reads "Ai9"

5a. Can do this manually in the nano file, make sure to save as a different name:
```
Ai9.fa
```

5b. Follow 10x directions: "There are special characters such as spaces in the header (all text after the >) of this FASTA sequence. 
These can be problematic for downstream applications. It can be helpful to change the header to be more informative and also to remove these characters. 
The following command opens the file and uses the stream editor (sed) function to search for a pattern (the original header), replace it with new text ("GFP"), 
then directs the output to a new output file, GFP.fa."

Synthetic construct tandem-dimer red fluorescent protein gene
cat GFP_orig.fa | sed s/L29345\.\1\ Aequorea\ victoria\ green\-fluorescent\ protein\ \(GFP\)\ mRNA\,\ complete\ cds/GFP/ > GFP.fa

for me: cat Ai9_tdT_orig.fa | sed s/AY678269\.\1\ Synthetic\ construct\ tandem\-dimer\ red\ fluorescent\ protein\ gene\,\ complete\ cds/Ai9/ > Ai9_tdT.fa

### 6. Make a custom GTF for Ai9:
6a. Find the number of bases in your sequence:
```
cat GFP.fa | grep -v "^>" | tr -d "\n" | wc -c
```
Use grep -v "^>" to search all lines that don't start with the > character, which removes line returns with tr -d "\n" so they aren't counted, and then counts the number of characters with the command wc -c. 
Each command is sent to the next step with the pipe | command.
Note: Ai9 has 1431 bases

6b. Incorporate that number into this code:
```
echo -e 'Ai9\tunknown\texon\t1\t1431\t.\t+\t.\tgene_id "Ai9"; transcript_id "Ai9"; gene_name "Ai9"; gene_biotype "protein_coding";' > Ai9.gtf
```
This command uses the function echo -e (prints everything in quotes; the -e enables interpretation of the backslash, e.g. \t). Use \t to insert the tabs that separate the 9 columns of information required for GTF.

The Ai9.GTF file should look something like this: 
GFP     unknown exon    1       922     .       +       .       gene_id "GFP"; transcript_id "GFP"; gene_name "GFP"; gene_biotype "protein_coding";

### 7. Append the new Ai9 FASTA file to the genome FASTA file:
7a. First copy the genome FASTA file, renaming it to include "Ai9" so you can differentiate:
```
cp genome.fa genome.Ai9tdt.fa
```
7b. Then append the file:
```
cat Ai9tdT.fa >> genome.Ai9tdT.fa
```

### 8. Append the new Ai9 GTF file to the genome GTF file:
8a. Then copy the genome GTF file, renaming it to include "Ai9" so you can differentiate:
```
cp genes.gtf genes.Ai9tdT.gtf
```
8b. Then append the file:
```
cat Ai9.gtf >> genes.Ai9tdT.gtf
```
8c. Check that the Ai9 gene is now present:
```
tail genes.Ai9tdT.gtf
```

# remember to keep the corresponding FASTA/GTF files in their respective folders. This directory organization gives the cellrangermkref inputs clarity and quick reference for yourself.

9. Create your job in nano:
```
nano run_CellRangermkref.sh
```
here's where my code stands:
```
  GNU nano 2.3.1                               File: run_CellRangermkref.sh                                                                      
#!/bin/bash
#SBATCH --job-name=CellRangermkref1
#SBATCH --output=/scratch/users/annie999/Cellrangermkref_output.txt
#SBATCH --error=/scratch/users/annie999/Cellrangermkref_error.txt
#SBATCH --partition=krasnow
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=10
#SBATCH --mem=64G
#SBATCH --time=48:00:00
#SBATCH --mail-user=annie999@stanford.edu
#SBATCH --mail-type=BEGIN,END,FAIL

# Load any necessary modules (if required)
module load cellranger/8.0.0

# Change to the scratch directory. This is critical, because the generated output will be quite large. Learned this the hard way.
cd /scratch/users/annie999

/oak/stanford/groups/krasnow/ATN/CIN1_CellRangerFiles/cellranger-8.0.0/cellranger mkref \
 --genome=MusmusculusAi9tdT_genome \
 --fasta=/oak/stanford/groups/krasnow/ATN/CIN1_CellRangerFiles/refdata-gex-GRCm39-2024-A/fasta/genome.Ai9tdT.fa \
 --genes=/oak/stanford/groups/krasnow/ATN/CIN1_CellRangerFiles/refdata-gex-GRCm39-2024-A/genes/genes.Ai9tdT.gtf \
 --localcores=10 \
 --localmem=64
```

10. Submit the job:
```
sbatch CellRangermkref.sh
```





 
