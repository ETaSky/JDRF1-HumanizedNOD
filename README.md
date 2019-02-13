# QIIME2 processing on JDRF1 Humanized NOD mice samples
**Initiation Date:** 2019-02-11

## Creating folders
```
mkdir -p 0-fastq
mkdir -p 1-imported_seqs
mkdir -p 2-demux
mkdir -p 3-dada
mkdir -p 4-div # folder for alpha and beta diversity analysis
```

## Renaming the raw data
To meet qiime2 requirement
```
cp 0-RawSeq/lane1_S1_L001_R1_001.fastq.gz 0-fastq/forward.fastq.gz
cp 0-RawSeq/lane1_S1_L001_I1_001.fastq.gz 0-fastq/barcodes.fastq.gz
cp 0-RawSeq/lane1_S1_L001_R2_001.fastq.gz 0-fastq/reverse.fastq.gz
```

## Import into QIIME2
```
qiime tools import \
    --type EMPPairedEndSequences \
    --input-path 0-fastq \
    --output-path 1-imported_seqs/imported_pe_seqs.qza
```

## Demultiplexing
```
qiime demux emp-paired --i-seqs 1-imported_seqs/imported_pe_seqs.qza --m-barcodes-file Mapping_JDRF1_HumanizedNOD-20190210-KM.txt --m-barcodes-column BarcodeSequence --o-per-sample-sequences 2-demux/demux.qza
qiime demux summarize \
    --i-data 2-demux/demux.qza \
    --o-visualization 2-demux/demux.qzv
# check sequencing results
```

## Denoising with dada2
```
qiime dada2 denoise-paired \
    --i-demultiplexed-seqs 2-demux/demux.qza \
    --p-trim-left-f 5 \
    --p-trim-left-r 5 \
    --p-trunc-len-f 150 \
    --p-trunc-len-r 150 \
    --o-table 3-dada/table.qza \
    --o-representative-sequences 3-dada/rep-seqs.qza \
    --o-denoising-stats 3-dada/denoising-stats.qza

## Summarizing feature table
qiime feature-table summarize \
    --i-table 3-dada/table.qza \
    --o-visualization 3-dada/table.qzv \
    --m-sample-metadata-file Mapping_JDRF1_HumanizedNOD-20190210-KM.txt

## Sumarizing representative sequences
qiime feature-table tabulate-seqs \
    --i-data 3-dada/rep-seqs.qza \
    --o-visualization 3-dada/rep-seqs.qzv

## Summarizing mapping file
qiime metadata tabulate \
	--m-input-file 3-dada/denoising-stats.qza \
	--o-visualization 3-dada/denoising-stats.qzv

## Build a phylogenetic tree for phylogeny based analysis
qiime phylogeny align-to-tree-mafft-fasttree \
    --i-sequences 3-dada/rep-seqs.qza \
    --o-alignment 3-dada/aligned-rep-seqs.qza \
    --o-masked-alignment 3-dada/masked-aligned-rep-seqs.qza \
    --o-tree 3-dada/unrooted-tree.qza \
    --o-rooted-tree 3-dada/rooted-tree.qza
```

## Taxanomy assignment

Here the most updated silva database will be used for taxanomy assignment. Silva version is v132, it was trained on scikit-learn v0.19.1, compatible to QIIME2-2018.8.

*Also it might need to set the temp file folder on the disk to prevent no space left on disk error, because the temp file is pretty large for this process* `export TMPDIR='/to/the/location/'`

```
qiime feature-classifier classify-sklearn --i-classifier ~/downloads/silva-132-99-515-806-nb-classifier-scikit-learn0.19.1.qza --i-reads 3-dada/rep-seqs.qza --o-classification 3-dada/taxonomy_silva132.qza --p-n-jobs 12 --verbose
```

## Further steps
1. Check the sequencing depth and decide rarifaction level for diversity analysis
2. Diversity analysis based on rarifaction
3. Taxanomy analysis based on rarifaction
