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
Metadata files stored in Google Sheets can be validated using Keemei ; an open-source Google Sheets plugin available at[Keemei](https://keemei.qiime2.org) 

Once Keemei is installed, in Google Sheets select Add-ons > Keemei > Validate QIIME 2 metadata file to determine whether the metadata file meets the required formatting of QIIME 2.
## Importing sequence files and metadata

## Quality control
