# VEME 2023 NGS Metagenomic classification Tutorial

## Carla Mavian

#### `https://github.com/cmavian/VEME_metagenomics`
#### 
#### 
### 1. Background and Metadata
We have collected field Aedes aegypti mosquitoes from different locations across the state of Florida, in the United
States, and we want to understand what is the composition of their microbiome. We have extracted RNA and
performed shotgun sequencing with Illumina. Our reads are 100bp single-read. You can read more about this study here: https://journals.asm.org/doi/10.1128/msphere.00316-20

#### Metagenomic Workflow: 
In this tutorial we will learn how to taxonomically classify and visualize our metagenomic reads obtained with Illumina using the following programs:

1. [Kraken2](https://ccb.jhu.edu/software/kraken2/index.shtml) [Manual](https://github.com/DerrickWood/kraken2/wiki/Manual)
2. [Bracken](https://ccb.jhu.edu/software/bracken/) (Bayesian Reestimation of Abundance with KrakEN) 
3. [Krona](https://github.com/marbl/Krona/wiki/KronaTools) [Wiki](https://github.com/marbl/Krona/wiki)


#### Setting up our folder for the analysis:

1. going into home directory

   ``cd``

2. let's make a working folder
 
   ``mkdir Desktop/metagenomics`` 

3. go in the folder and make a link to the files we are using

    `` cd Desktop/metagenomics`` 
    `` ln -s /usr/local/share/data/metagenomics/data/*.fastq.gz . `` 



#### 
#### 
### 2. Kraken2: 
Kraken is a taxonomic sequence classifier that assigns taxonomic labels to DNA sequences. 
Kraken examines the k-mers within a query sequence and uses the information within those k-mers to query a database. That database maps k-mers to the lowest common ancestor (LCA) of all genomes known to contain a given k-mer.

To get a full list of options, use kraken2 --help.

#### Standard Kraken Output Format
* --use-names replaces the taxonomy ID column with the scientific name and the taxonomy ID in parenthesis (e.g., "Bacteria (taxid 2)"). 
* --report option writes out the sample report in tab-delimited format

Kraken 2's output contains:
1. "C"/"U": a one letter code indicating that the sequence was either classified or unclassified. A rank code, indicating (U)nclassified, (R)oot, (D)omain, (K)ingdom, (P)hylum, (C)lass, (O)rder, (F)amily, (G)enus, or (S)pecies. 
2. The sequence ID, obtained from the FASTA/FASTQ header.
3. The taxonomy ID Kraken 2 used to label the sequence; this is 0 if the sequence is unclassified.
4. The length of the sequence in bp.
5. A space-delimited list indicating the LCA mapping of each k-mer in the sequence.


#### Running Kraken
   ``kraken2 --use-names --db $KRAKEN_DB seqs.fa --report kreport``

### Running a for loop on all fastq files for the code:
	
    ``for infile in *.fastq.gz
        do
        	base=${infile%.fastq.gz}
            kraken2 --use-names --db /data/kraken2/minikraken2_v1_8GB --report ${base}.kreport ${infile}
        done``  
	
#### 
#### 
### 3. Bracken:
Bracken (Bayesian Reestimation of Abundance with KrakEN) is a highly accurate statistical method that computes the abundance of species in DNA sequences from a metagenomics sample. 
Braken uses the taxonomy labels assigned by Kraken2 to estimate the number of reads originating from each species present in a sample.

#### Bracken Output File Format
Bracken output file; tab-delimited columns

Name
Taxonomy ID
Level ID (S=Species, G=Genus, O=Order, F=Family, P=Phylum, K=Kingdom)
Kraken Assigned Reads
Added Reads with Abundance Reestimation
Total Reads after Abundance Reestimation
Fraction of Total Reads

#### Running Bracken
Bracken can be run using either the bracken shell script or the est_abundance python script. 

``bracken -i input.kreport -d $KRAKEN_DB/ -r ${READ_LEN} -o output.bracken``

${READ_LEN} default =  100

Our reads are 1x100bp so we do not need to speify. If you are using the pyhon script, you need to speficy the kmer distribution database corresponding to the Kraken database previously used and your read length.  

python script alternative:
   ``python est_abundance.py -i input.kreport -k $KRAKEN_DB/database$READ_LENmers.kmer_distrib -o output.bracken``

### Running a for loop on all fastq files for the code:
	
    ``for infile in *.kreport
        do
        	base=${infile%.kreport}
            /usr/local/share/Bracken-2.7/bracken -i ${infile} -d /data/kraken2/minikraken2_v1_8GB/ -o ${base}.bracken 
        done`` 


#### 
#### 
### 4. KRONA

#### Report convertion 
kreport2krona.py converts a Kraken-style report into Krona-compatible format (https://github.com/jenniferlu717/KrakenTools/blob/master/kreport2krona.py)

   ``python kreport2krona.py -r mysample_bracken.kreport -o mysample.krona``

### Running a for loop on all fastq files for the code:

    ``for infile in *_bracken_species.kreport
        do
			base=${infile%_bracken_species.kreport}
			python /usr/local/share/kreport2krona.py -r ${infile} -o ${base}.krona
        done``

#### Krona chart
Use ktImportText to create a chart based on a txt file that lists values and wedge hierarchies to add them to (https://github.com/marbl/Krona/wiki/Importing-text-and-XML-data)

   ``ktImportText mysample.krona -o mysample.html ``

### Running a for loop on all fastq files for the code:

    ``for infile in *.krona
        do
			base=${infile%.krona}
			ktImportText ${infile} -o ${base}.html
        done``



