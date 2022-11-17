# Microbiome analysis using QIIME2

## Data
A subset data from the Early Childhood Antibiotics and the Microbiome (ECAM) study will be used.The study tracked microbiome composition and development of 43 infants in the United States from birth to two years of age.
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
The metadata contains information on: [age at the time of collection, birth mode and diet of the child, the type of DNA sequencing]
In QIIME2 metadata is usually stored as a TSV file (Tab separated file format).QIIME has a metadata formating requirement that can be checked using Keemei (Rideout et al., 2016)
#### Validating metadata
Metadata files stored in Google Sheets can be validated using Keemei ; an open-source Google Sheets plugin available at [Keemei](https://keemei.qiime2.org) 

Once Keemei is installed, in Google Sheets select Add-ons > Keemei > Validate QIIME 2 metadata file to determine whether the metadata file meets the required formatting of QIIME 2.
## Importing sequence files and metadata

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
## Taxonomic classification
## Diversity analysis
## Phologenetic tree construction

