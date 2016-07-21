# Rn5_mappability (by Yu Wang, 02/02/2016).

# Summary for Mappability file for Rat Genome(rn5)

In our rat project, with the help of NGS sequencing, we aimed to establish a SNP Array Panel for Rat genotyping. After the design of SNP array, the first thing is evaluation of SNP Array density. This procedure need the help of mapping ability file of rn5, and turn out there is no available mappability file for rat. In the annotation of rn5 from UCSC, they have no record for telomere and centromere. 

In this repository, I make a note for how to generate mappability files, both with GEM package and my own tools. I adpoted the method of GEM for mapping score (Derrien et al. 2012, http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0030377) and the whole procedure followed the instruction of BITS (http://wiki.bits.vib.be/index.php/Create_a_mappability_track#create_a_.27gem.27_index). The real mapping ability for pair end reads is a little higher, because GEM only consider single end reads.

# Preparation for Dataset and Softwares.
- download reference from UCSC (http://hgdownload.soe.ucsc.edu/goldenPath/rn5/bigZips/rn5.fa.gz); 

- install GEM package following these instructions: http://algorithms.cnag.cat/wiki/The_GEM_library;

- install tabix from Heng Li, which is useful for quick jump from location to location. (http://www.htslib.org/doc/tabix.html)

# Command line (take 100bp read for example):
gem-indexer -T 4 -c dna -i rn5.fa -o rn5.index # index reference with 4 processors

gem-mappability -T 8 -I rn5.index.gem -l 100 -o rn5_100  # create mappability raw data, ASCII Score for each site, take 1-2 hours.

gem-2-wig -I rn5.index.gem -i rn5_100.mappability -o rn5_100 # create wig file based on raw ASCII Score

./BigWig2Bed.pl rn5_100.wig | bgzip -c > rn5_100.wig.mapQ.gz # use inhouse tool, covert BigWig format to Bed and bgzip for tabix index.

tabix -p bed rn5_100.wig.mapQ.gz # index genome location with tabix

# How to use these files
Get mappability of region 1:2000-3000:
tabix rn5_100.wig.mapQ.gz 1:2000-3000 > rn5_100.wig.mapQ.seg

Get mappability of whole chromosome 1 and draw a figure for that:
tabix rn5_100.wig.mapQ.gz 1 | ./SlideWinDepth.pl -w 1000000 -s 0 -c1 -p2 -d4 -b - > rn5_100.chr1.slide

# File Description:
rn5_100.wig.mapQ.gz: includes mappability for each segment, read length: 100bp,  Bed format with 1-based coordinate, format: chr, pos_start, pos_end, mapping_score.

BigWig2Bed.pl: Script for converting BigWig format to BED format;

SlideWinDepth.pl: Script for sliding window statistics, firstly design for pileup, now available for BED file. 
