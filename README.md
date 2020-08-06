## Codes and functions for the multiscale analysis of the *Escherichia coli* 3C data ##

This page presents the different codes and functions developed to perform the analysis described in the article **Multiscale structuring of the *E. coli* chromosome by nucleoid-associated and condensin proteins** by Virginia S. Lioy, Axel Cournac, et al. The codes presented here should allow to reproduce the different graphs and figures from the main text and the supplementary data. 

### Table of contents

* [Dependencies](https://github.com/koszullab/E.coli.analysis/blob/master/README.md#dependencies)
* [Raw data extraction and alignment](https://github.com/koszullab/E.coli.analysis/blob/master/README.md#raw-data-extraction-and-alignment)
* [Building contacts map](https://github.com/koszullab/E.coli.analysis/blob/master/README.md#building-contacts-map)
* [Correlation with other data](https://github.com/koszullab/E.coli.analysis/blob/master/README.md#correlation-with-other-data)
* [Scalogram visualization tool](https://github.com/koszullab/E.coli.analysis/blob/master/README.md#scalogram-vizulaisation-tool)
* [Directionality Index](https://github.com/koszullab/E.coli.analysis/blob/master/README.md#directionality-index)
* [Correlation between transcription and contacts](https://github.com/koszullab/E.coli.analysis/blob/master/README.md#correlation-between-transcription-and-contacts)
* [3D structure](https://github.com/koszullab/E.coli.analysis/blob/master/README.md#3d-structure)
* [Ratio of contacts](https://github.com/koszullab/E.coli.analysis/blob/master/README.md#ratio-of-contacts)


### Dependencies

Scripts and codes can be run on OS X and other Unix-based systems, and necessitate:
#### *Python (>=2.7)*
* Numpy
* Matplotlib (>=1.0)
* Scipy
* Biopython

#### External programs

* `Bowtie2 ` / [bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml)
* `Pymol` / [Pymol](https://www.pymol.org/)
* `R` / [R](https://cran.r-project.org/)
* `Shrec3D` / [Shrec3D](https://sites.google.com/site/julienmozziconacci/)



## Raw data extraction and alignment
#### Data extraction

FASTQ files of the reads were deposited in the NCBI database under the GEO accession number GSE107301. A SRA executable called fastq-dump from SRA can be used to extract and split both reads of pair-end sequences: 
```bash
fastq-dump library_identification --split-3 -O /path_to_a_directory
```


#### Alignment

We use the MG1655 reference genome (GenBank: U00096.2, total length 4639675), the program Bowtie2 and an iterative procedure (see for instance the one described in [DADE] (https://github.com/scovit/dade). We process the pairs of reads so only read with a mapping quality > 30 are retained. For instance:
```bash
#  Keeping only the columns of the sam file that contain necessary information:
awk '{print $1,$3,$4,$2,$5;}' p1.sam > p1.sam.0
awk '{print $1,$3,$4,$2,$5;}' p2.sam > p2.sam.0

# Sort according to the read identification to have both mates in the same order
# if sort does not have -V option try -d
sort -V -k1 p1.sam.0 > p1.sam.0.sorted
sort -V -k1 p2.sam.0 > p2.sam.0.sorted

# Pairing of both mates in a single file
paste p1.sam.0.sorted p2.sam.0.sorted > p1_p2_merged

# Removal of intermediar files
rm p1.sam.0.sorted
rm p2.sam.0.sorted

# Filtering of pairs of reads that both have a Mapping Quality above 30
awk '{if($1 eq $6 && $5>= 30 && $10 >= 30) print $2,$3,$4,$7,$8,$9}'  p1_p2_merged  > output_alignment_idpt.dat

# Removal of intermediar file
rm p1_p2_merged
```
At this stage, you have a file (output_alignment_idpt.dat.ind3) containing these information organized as such:
```
chr1 104180 16 chr1 104057 0
chr1 3570510 16 chr1 3570450 0
chr1 4255981 0 chr1 4256104 16
chr1 159457 16 chr1 159370 0
chr1 4113710 16 chr1 4113584 0
chr1 4259849 16 chr1 4259818 0
chr1 3753874 0 chr1 3754001 16
chr1 2856270 16 chr1 2856124 0
chr1 4134782 16 chr1 4134678 0
```

where chr1 corresponds to *Escherichi coli* genome. Each read is assigned to a restriction fragment as described in https://github.com/axelcournac/3C_tutorial (Cournac et al., MMiB, 2016).


## Building contacts map
To build the contact map and filter non-informative events, we use 3Cevents_MATRICE.py [`fragment_attribution.py`](python_codes/fragment_attribution.py):
```bash

python fragment_attribution.py /media/axel/RSG3/BACK_UP/axel/Bureau/python/fasta/ecoli/ HpaII output_alignment_idpt_BC76.dat

python library_events.py output_alignment_idpt_BC76.dat.indices BC76_MatP

python Matrice_Creator.py output_alignment_idpt_BC76.dat.indices.filtered 5000 BC76_MatP_37C
```
The first argument corresponds to the path of the file “output_alignment_idpt.dat.ind3”, which contains the alignment information. The second argument corresponds to the size of the bin (here: 5,000bp). The third argument is the name of the prefix for the file of the contacts maps. To plot and/or save the contact map, use [plot_mat_temp.py](python_codes/plot_mat_temp.py)
```bash
python plot_mat_temp.py mat_temp_WT_rep1_5000.txt WT_rep1_5000
```
![alt tag](https://github.com/koszullab/E.coli.analysis/blob/master/pictures/Ecolichromosomemap.png)