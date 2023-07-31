# VEME 2023 NGS Metagenomic classification Tutorial

## Carla Mavian
### `https://github.com/cmavian/VEME_metagenomics`
#### 
#### 
### 1. Background and Metadata
We have collected field Aedes aegypti mosquitoes from different locations across the state of Florida, in the United
States, and we want to understand what is the composition of their microbiome. We have extracted RNA and
performed shotgun sequencing with Illumina.

#### In this tutorial we will learn how to taxonomically classify and visualize our metagenomic reads obtained with Illumina using the following programs:

1. [Kraken2](https://ccb.jhu.edu/software/kraken2/index.shtml) [Manual](https://github.com/DerrickWood/kraken2/wiki/Manual)
2. [Bracken](https://ccb.jhu.edu/software/bracken/) (Bayesian Reestimation of Abundance with KrakEN) 
3. [Krona](https://github.com/marbl/Krona/wiki/KronaTools) [Wiki](https://github.com/marbl/Krona/wiki)

#### Metagenomic Workflow 

<figure>
    <img src="XXXXXX.png" width="230" height="300">
    <figcaption>Metagenomic Workflow</figcaption>
</figure>

#### 
#### 
### 2. Kraken2: 
To get a full list of options, use kraken2 --help.

#### Kraken is a taxonomic sequence classifier that assigns taxonomic labels to DNA sequences. 
#### Kraken examines the k-mers within a query sequence and uses the information within those k-mers to query a database. That database maps k-mers to the lowest common ancestor (LCA) of all genomes known to contain a given k-mer.
#### Features of Kraken2:
1. Only minimizers of the k-mers in the query sequences are used as database queries. 
2. Kraken2 uses a compact hash table that is a probabilistic data structure. This means that occasionally, database queries will fail by either returning the wrong LCA, or by not resulting in a search failure when a queried minimizer was never actually stored in the database. Users should be aware that database false positive errors occur in less than 1% of queries, and can be compensated for by use of confidence scoring thresholds.
3. Kraken2 has the ability to build a database from amino acid sequences and perform a translated search of the query sequences against that database.
4. Kraken2 utilizes spaced seeds in the storage and querying of minimizers to improve classification accuracy.
5. Kraken2 provides support for "special" databases that are not based on NCBI's taxonomy. These are currently limited to three popular 16S databases. 
6. You can also build your own custom database.


#### Standard Kraken Output Format
Through the use of kraken2 --use-names, Kraken2 will replace the taxonomy ID column with the scientific name and the taxonomy ID in parenthesis (e.g., "Bacteria (taxid 2)" instead of "2"). The sample report functionality now exists as part of the kraken2 script, with the use of the --report option; the sample report formats are described below.

Each sequence (or sequence pair, in the case of paired reads) classified by Kraken 2 results in a single line of output. Kraken 2's output lines contain five tab-delimited fields; from left to right, they are:

"C"/"U": a one letter code indicating that the sequence was either classified or unclassified.

The sequence ID, obtained from the FASTA/FASTQ header.

The taxonomy ID Kraken 2 used to label the sequence; this is 0 if the sequence is unclassified.

The length of the sequence in bp. In the case of paired read data, this will be a string containing the lengths of the two sequences in bp, separated by a pipe character, e.g. "98|94".

A space-delimited list indicating the LCA mapping of each k-mer in the sequence(s). For example, "562:13 561:4 A:31 0:1 562:3" would indicate that:

* the first 13 k-mers mapped to taxonomy ID #562
* the next 4 k-mers mapped to taxonomy ID #561
* the next 31 k-mers contained an ambiguous nucleotide
* the next k-mer was not in the database
* the last 3 k-mers mapped to taxonomy ID #562
* Note that paired read data will contain a "|:|" token in this list to indicate the end of one read and the beginning of another.

When Kraken 2 is run against a protein database (see [Translated Search]), the LCA hitlist will contain the results of querying all six frames of each sequence. Reading frame data is separated by a "-:-" token.


#### Confidence scoring:
The approach we use allows a user to specify a threshold score in the [0,1] interval; the classifier then will adjust labels up the tree until the label's score (described below) meets or exceeds that threshold. If a label at the root of the taxonomic tree would not have a score exceeding the threshold, the sequence is called unclassified by Kraken 2 when this threshold is applied.

A sequence label's score is a fraction C/Q, where C is the number of k-mers mapped to LCA values in the clade rooted at the label, and Q is the number of k-mers in the sequence that lack an ambiguous nucleotide (i.e., they were queried against the database). Consider the example of the LCA mappings in Kraken 2's output given earlier:

For the example above, "562:13 561:4 A:31 0:1 562:3", a label of #562 for this sequence would have a score of C/Q = (13+3)/(13+4+1+3) = 16/21. 
A label of #561 would have a score of C/Q = (13+4+3)/(13+4+1+3) = 20/21. 
If a user specified a --confidence threshold over 16/21, the classifier would adjust the original label from #562 to #561; if the threshold was greater than 20/21, the sequence would become unclassified.


#### Sample Report Output Format
Like Kraken 1, Kraken 2 offers two formats of sample-wide results. Kraken 2's standard sample report format is tab-delimited with one line per taxon. The fields of the output, from left-to-right, are as follows:

Percentage of fragments covered by the clade rooted at this taxon
Number of fragments covered by the clade rooted at this taxon
Number of fragments assigned directly to this taxon
A rank code, indicating (U)nclassified, (R)oot, (D)omain, (K)ingdom, (P)hylum, (C)lass, (O)rder, (F)amily, (G)enus, or (S)pecies. Taxa that are not at any of these 10 ranks have a rank code that is formed by using the rank code of the closest ancestor rank with a number indicating the distance from that rank. E.g., "G2" is a rank code indicating a taxon is between genus and species and the grandparent taxon is at the genus rank.
NCBI taxonomic ID number
Indented scientific name
The scientific names are indented using space, according to the tree structure specified by the taxonomy.


#### Running Kraken
* kraken2 --use-names --db $KRAKEN_DB seqs.fa --report kreport

### Running a for loop on all fastq files for the code:
	
    ``$ for infile in *.fastq.gz
        do
        	base=${infile%.fastq.gz}
            kraken2 --use-names --db /data/kraken2/minikraken2_v1_8GB --report ${base}.kreport ${infile}
        done``  
	
#### 
#### 
### 3. Bracken:
#### Bracken (Bayesian Reestimation of Abundance with KrakEN) is a highly accurate statistical method that computes the abundance of species in DNA sequences from a metagenomics sample. 
#### Braken uses the taxonomy labels assigned by Kraken, a highly accurate metagenomics classification algorithm, to estimate the number of reads originating from each species present in a sample. 
#### Kraken classifies reads to the best matching location in the taxonomic tree, but does not estimate abundances of species. We use the Kraken database itself to derive probabilities that describe how much sequence from each genome is identical to other genomes in the database, and combine this information with the assignments for a particular sample to estimate abundance at the species level, the genus level, or above. 
#### Combined with the Kraken classifier, Bracken produces accurate species- and genus-level abundance estimates even when a sample contains two or more near-identical species.


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
#### Bracken can be run using either the bracken shell script or the est_abundance python script. 
#### We are running the python script that requires specification of the exact path to the database files. 
#### Our reads are 1x100bp so we are using the kmer distribution database corresponding to the Kraken database we used and our read length.  

* python est_abundance.py -i input.kreport -k $KRAKEN_DB/database$READ_LENmers.kmer_distrib -o output.bracken

### Running a for loop on all fastq files for the code:
	
    ``$ for infile in *.kreport
        do
        	base=${infile%.kreport}
            python src/est_abundance.py -i ${infile} -k /data/kraken2/minikraken2_v1_8GB/database100mers.kmer_distrib -o ${base}.bracken 
        done`` 


#### 
#### 
### 4. KRONA

#### kreport2krona.py converts a Kraken-style report into Krona-compatible format (https://github.com/jenniferlu717/KrakenTools/blob/master/kreport2krona.py)
* python kreport2krona.py -r mysample_bracken.kreport -o mysample.krona

### Running a for loop on all fastq files for the code:

    ``$ for infile in *_bracken.kreport
        do
			base=${infile%_bracken.kreport}
			python kreport2krona.py -r ${infile} -o ${base}.krona
        done``

#### Use ktImportText to create a chart based on a txt file that lists values and wedge hierarchies to add them to (https://github.com/marbl/Krona/wiki/Importing-text-and-XML-data)
* ktImportText mysample.krona -o mysample.html

### Running a for loop on all fastq files for the code:

    ``$ for infile in *.krona
        do
			base=${infile%_krona}
			ktImportText ${infile} -o ${base}.html
        done``



<figure>
    <img src="XXXXXX.png" width="230" height="300">
    <figcaption>Metagenomic Workflow</figcaption>
</figure>






