# Microbiome analysis using QIIME2
QIIME 2 is a powerful microbiome analysis package with a focus on data and analysis transparency. QIIME 2 enables researchers to start an analysis with raw DNA sequence data and finish with publication-quality figures and statistical results.
QIIME2 is a redesign of QIIME1.It supports end-to-end analysis of microbiome data.
QIIME2 analysis vary depending on your experimental and data analysis goal.

* Bionformatics workflow

## Data
A subset data from the Early Childhood Antibiotics and the Microbiome (ECAM) study((Bokulich,Chung, et al., 2016) will be used.The study tracked microbiome composition and development of 43 infants in the United States from birth to two years of age.

- Sample: Infant fecal sample
- Sequence: V4 region of 16s rRNA gene
- Sequencing platform: Illumina MiSeq
## Downloading sample data and Metadata 
```
mkdir QIIME2-tutorial
cd QIIME2-tutorial
wget -O 81253.zip https://qiita.ucsd.edu/public_artifact_download/?artifact_id=81253
unzip 81253.zip
mv mapping_files/81253_mapping_file.txt metadata.tsv
```
### Metadata files
In microbiome studies sample metadata describes the sample data.It may include information on sample collection site,time,how they were processed and analyzed.To explore the ECAM study metadata let’s check the first 10 lines.
```
head metadata.tsv
```
The metadata may contain information on: [age at the time of collection, birth mode and diet of the child, the type of DNA sequencing]
In QIIME2 metadata is usually stored as a TSV file (Tab separated file format).QIIME has a metadata formating requirement that can be checked using Keemei (Rideout et al., 2016)
#### Validating metadata
Metadata files stored in Google Sheets can be validated using Keemei ; an open-source Google Sheets plugin available at [Keemei](https://keemei.qiime2.org) 

Once Keemei is installed, in Google Sheets select Add-ons > Keemei > Validate QIIME 2 metadata file to determine whether the metadata file meets the required formatting of QIIME 2.
## Importing data

Data used in QIIME exist as QIIME artifacts and have a *.qza* extension with the exception of the metadata.
Artifacts are zip files containing data and QIIME2 specific metadata.

QIIME2 allows you to import and export data at any step of the analysis.Before importing the data check whether its paired-end,single-end,multiplexed or demultiplexed.
Multiplexed data comes as Forward and Reverse reads while demultiplexed occur as one sequence file per sample.

To import files into QIIME we will need to create a manifest file.A manifest file is a tab separated file containing a sample_ID and absolute path.
### Creating a manifest file
```
# Add the column header (sample-id and absolute-filepath) to the manifest file
echo -e "sample-id\tabsolute-filepath" > manifest.tsv

# Using a for loop to add sample-id and absolute path to manifest file.
for f in 'ls per_sample_FASTQ/81253/*.gz'; do
n='basename $f';
echo -e "12802.${n%.fastq.gz}\t$PWD/$f"; done >>
manifest.tsv
```
### Importing data 
```
qiime tools import \
--input-path manifest.tsv \
--type 'SampleData[SequencesWithQuality]' \
--input-format SingleEndFastqManifestPhred33V2 \
--output-path se-demux.qza
```
## Quality control
Demultiplex artifacts allows you to create an interactive summary of sequences that allows us to assess the overall quality of our reads.
### Creating summary of demultiplexed artifact
```
qiime demux summarize \
--i-data se-demux.qza \
--o-visualization se-demux.qzv
```
The summary provides information on number of sequences per sample and the distribution of sequence quality score at each position.
The output ia a .qzv file that can be viewed via the browser by running the following code.
```
qiime tools view se-demux.qzv
```
### Denoising
Denoising involves correction of Amplicon sequence errors.QIIME2 offers denoising via DADA2 and Deblur.
Deblur uses a precalculated static sequence error profile to associate erroneous sequence reads with true biological sequence from which they were derived. 
DADA2 creates sequence error profiles on per sample analysis basis.Deblur has a shorter run time compared to DADA2.
### Deblur
Deblur denoising is executed in two steps

*Step One : Initial quality filtering based on quality scores
```
qiime quality-filter q-score \
--i-demux se-demux.qza \
--o-filtered-sequences demux-filtered.qza \
--o-filter-stats demux-filter-stats.qza
```
*Step Two : Apply the Deblur workflow using the denoise-16S action
```
qiime deblur denoise-16S \
--i-demultiplexed-seqs demux-filtered.qza \
--p-trim-length 150 \
--p-sample-stats \
--p-jobs-to-start 4 \
--o-stats deblur-stats.qza \
--o-representative-sequences rep-seqs-deblur.qza \
--o-table table-deblur.qza
```
--p-trim-length  option truncates the sequence at position n.The parameter is determined from the interactive plots above.It is recommended to set this value to a length where the median quality score begins to drop below 30, or 20 if the overall run quality is too low.

* Visualization of Deblur statistics
```
qiime deblur visualize-stats \
--i-deblur-stats deblur-stats.qza \
--o-visualization deblur-stats.qzv
```
* Visualizing rep-sequences
```
qiime feature-table tabulate-seqs \
--i-data rep-seqs-deblur.qza \
--o-visualization rep-seqs-deblur.qzv
```
* Visualizing feature table

At this step you can add metadata file ;which adds information  about sample groups into the summary output.Adding metadata is useful for checking that all groups have enough samples and sequences before proceeding with the downstream analysis.
```
qiime feature-table summarize \
--i-table table-deblur.qza \
--m-sample-metadata-file metadata.tsv \
--o-visualization table-deblur.qzv
```
## Phylogenetic tree construction
Microbiome data can analyzed without a phylogenetics tree.However,some diversity analysis methods like Unifrac require one.
Phylogenetic tree allows us to consider evolutionary relatedness between the DNA sequences.
For this tutorial,we will use the fragment-insertion tree-building method using the sepp action of the q2-fragment-insertion plugin.
This method aligns our unknown short fragments to full-length sequences in a known reference database and then places them onto a fixed tree.
* Download a backbone tree as the base for our features to be inserted onto.
```
#Download Greengenes (16s rRNA) reference database:
wget \
-O "sepp-refs-gg-13-8.qza" \
"https://data.qiime2.org/2022.8/common/sepp-refs-gg-13-8.qza"
```
* Create an insertion tree by entering the following commands:
```
qiime fragment-insertion sepp \
--i-representative-sequences rep-seqs-deblur.qza \
--i-reference-database sepp-refs-gg-13-8.qza \
--p-threads 4 \
--o-tree insertion-tree.qza \
--o-placements insertion-placements.qza
```
The newly formed insertion-tree.qza is stored as a rooted phylogenetic tree.Once the insertion tree is created, you must filter your feature table so that it only
contains fragments that are in the insertion tree to avoid diversity computational failure.
* Filter the feature table
```
qiime fragment-insertion filter-features \
--i-table table-deblur.qza \
--i-tree insertion-tree.qza \
--o-filtered-table filtered-table-deblur.qza \
--o-removed-table removed-table.qza
```
This command generates two feature tables: 

*The filtered-tabledeblur.qza* - contains only features that are also present in the tree, 

*removed-table.qza* - contains features not present in the tree.

* Visualize the phylogenetic tree

The phylogenetic tree artifact produced can be readily visualized using [q2-empress](https:// github.com/biocore/empress)  
or iTOL’s (Letunic & Bork, 2019) interactive web-based tool by simply uploading the artifact at [itol](https:// itol.embl.de/ upload.cgi.) 

## Taxonomic classification

QIIME 2 provides several methods to predict the most likely taxonomic affiliation of our features through the q2-feature-classifier plugin.We will use a Naive Bayes classifier,which must be trained on taxonomically defined reference sequences covering the target region of interest.
### Bespoke method of training a classifier
This method requires three files;reference reads, a reference taxonomy, and taxonomic weights.
* Downloading required files
```
git clone https://github.com/BenKaehler/readytowear.git
```

* Training a classifier

```
qiime feature-classifier fit-classifier-naive-bayes \
  --i-reference-reads readytowear/data/gg_13_8/515f-806r/ref-seqs.qza \
  --i-reference-taxonomy readytowear/data/gg_13_8/515f-806r/ref-tax.qza \
  --i-class-weight readytowear/data/gg_13_8/515f-806r/human-stool.qza \
  --o-classifier gg138_v4_human-stool_classifier.qza
```

* Performing Taxonomic classification
```
qiime feature-classifier classify-sklearn \
--i-reads rep-seqs-deblur.qza \
--i-classifier gg138_v4_human-stool_classifier.qza \
--o-classification bespoke-taxonomy.qza
```
* Visualization of taxonomic bars
```
qiime metadata tabulate \
--m-input-file bespoke-taxonomy.qza \
--m-input-file rep-seqs-deblur.qza \
--o-visualization bespoke-taxonomy.qzv
```
* Generate a taxonomic bar plots
```
qiime taxa barplot \
--i-table table-deblur.qza \
--i-taxonomy bespoke-taxonomy.qza \
--m-metadata-file metadata1.tsv \
--o-visualization se-bar-plots.qzv
```
## Diversity analysis
Diversity analysis can be performed in R or QIIME2.


















