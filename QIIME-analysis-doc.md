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
```
