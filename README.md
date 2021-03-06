## Getting Started

```sh
# Install hifiasm (requiring g++ and zlib)
git clone https://github.com/chhylp123/hifiasm
cd hifiasm && make
# Assembly
./hifiasm -o NA12878.asm -t 32 NA12878.fq.gz
```

## Introduction

Hifiasm is a fast haplotype-resolved de novo assembler for PacBio
Hifi reads. Unlike most existing assemblers, hifiasm starts from uncollapsed
genome. Thus, it is able to keep the haplotype information as much as possible.
The input of hifiasm is the PacBio Hifi reads in fasta/fastq format, and its
outputs consist of: 

1. Haplotype-resolved raw [unitig][unitig] graph in [GFA][gfa] format
   (*prefix*.r\_utg.gfa). This graph keeps all haplotype information, including
   somatic mutations and recurrent sequencing errors.
2. Haplotype-resolved processed unitig graph without small bubbles
   (*prefix*.p\_utg.gfa). This is usually the preferred output for highly
   heterozygous genomes.
3. Primary assembly [contig][unitig] graph (*prefix*.p\_ctg.gfa). This is the
   preferred output for inbred strains or human. For highly heterozygous
   genomes, this graph may represent multiple haplotypes. We plan to change
   this to represent one set of haplotypes.
4. Alternate assembly contig graph (*prefix*.a\_ctg.gfa).
5. Haplotype-aware error corrected reads in fasta format (*prefix*.ec.fa).
6. All-to-all overlaps in the [PAF][paf] format (*prefix*.ovlp.paf).

So far hifiasm is still in early development stage, it will output phased
chromosome-level high-quality assembly in the near future. In addition, hifiasm
also outputs three binary files that save all overlap inforamtion
(hifiasm.asm.ovlp, hifiasm.asm.ovlp.source, hifiasm.asm.ovlp.reverse in default). With these files, hifiasm can avoid the time-consuming all-to-all overlap calculation step, and do the assembly
directly and quickly. This might be helpful when you want to get an optimized
assembly by multiple rounds of experiments with different parameters.

Hifiasm is a standalone and lightweight assembler, which does not need external
libraries (except zlib). For large genomes, it can generate high-quality
assembly in a few hours. Hifiasm has been tested on the following datasets:

|<sub>Dataset<sub>|<sub>GSize<sub>|<sub>Cov<sub>|<sub>Asm options<sub>|<sub>CPU time<sub>|<sub>Wall time<sub>|<sub>RAM<sub>|<sub>[unitig][unitig]/[contig][unitig] N50<sup>[1]</sup><sub>|
|:---------------|-----:|-----:|:---------------------|-------:|--------:|----:|----------------:|
|<sub>[Human NA12878]<sub>|<sub>3Gb<sub>|<sub>x28<sub>|<sub>-k 40 -t 42 -r 2<sub>|<sub>200h<sub>|    <sub>5h32m<sub>|<sub>114G<sub>|<sub>93.5Kb/28.2Mb<sub>|
|<sub>[Human HG002]<sub>|<sub>3Gb<sub>|<sub>x43<sub>|<sub>-k 40 -t 42 -r 2<sub>|<sub>405h10m<sub>|<sub>12h7m<sub>|<sub>146G<sub>|<sub>320kb/35.3Mb<sub>|
|<sub>[Human CHM13]<sub>|<sub>3Gb<sub>|<sub>x27<sub>|<sub>-k 40 -t 42 -r 2<sub>|<sub>157h28m<sub>|<sub>5h10m<sub>|<sub>85.8G<sub>|<sub>NA<sup>[2]</sup>/41.4Mb<sub>|
|<sub>[Butterfly]<sub>|<sub>358Mb<sub>|<sub>x35<sub>|<sub>-k 40 -t 42 -r 2 -z 20<sub>|<sub>17h6m<sub>|<sub>36m<sub>|<sub>16G<sub>|<sub>7.5Mb/NA<sup>[3]</sup><sub>|
|<sub>[\[Redwood\]](https://downloads.pacbcloud.com/public/dataset/redwood2020/)<sub>|<sub>26.5Gb<sub>|<sub>x23<sub>|<sub>-k 40 -t 64 -r 2<sub>|<sub>7274h30m<sub>|<sub>141h30m<sub>|<sub>512G<sub>|<sub>1.7Mb/1.9Mb<sub>|

<sub>[1] unitig N50 is the N50 of assembly graph with haplotype information (i.e., bubbles), while the contig N50 is the N50 of haplotype collapsed assembly (i.e., without bubbles).
[2] CHM13 is a homozygous sample, so that unitig N50 makes no sense.
[3] Butterfly has high heterozygous rate, so that most chromosomes have been fully separated into two haplotypes. In this case, contig N50 makes no sense.<sub>

Note that different species need different assembly graphs. For homozygous genomes (i.e., Human CHM13), the primary assembly contig graph is the best choice. 
For species with high heterozygous rate (i.e., Butterfly), different haplotypes can be fully separated. It is important to remove small bubbles from the haplotype-resolved unitig graph. The
reason is that some small bubbles are caused by somatic mutations or noise in data, which are not
the real haplotype information. In this case, haplotype-resolved processed unitig graph
without small bubbles should be better. For ordinary human genome (i.e., Human NA12878 and HG002), different haplotypes cannot be fully separated due to the low heterozygous rate. There are many small bubbles including haplotype information, which cannot be simply removed. Thus, it is necessary to use the haplotype-resolved raw unitig graph. **Hifiasm will generate a universal haplotype-resolved contig graph for all species in the near future.**

## Usage

For Hifi reads assembly, a typical command line looks like:

```sh
./hifiasm -o NA12878.asm -t 32 NA12878.fq.gz
```

where `NA12878.fq.gz` is the input reads and `-o` specifies the output files.
In this example, all output files can be found at `NA12878.asm.*`. `-k`, `-t`
and `-r` specify the length of k-mer, the number of CPU threads, and the number
of correction rounds, respectively. Note that at first run, hifiasm will save
all overlaps to disk, which can avoid the time-consuming all-to-all overlap
calculation next time. For hifiasm, once the overlap information has been
obtained during the previous run in advance, it is able to load all overlaps
from disk and then directly do assembly. If you want to ignore the pre-computed
overlap information, please specify `-i`.

Please note that some old Hifi reads may consist of short adapters. To improve
the assembly quality, adapters should be removed by `-z` as follow:

```sh
./hifiasm -o butterfly.asm -t 42 -z 20 butterfly.fq.gz
```

In this example, hifiasm will remove 20 bases from both ends of each read.

[unitig]: http://wgs-assembler.sourceforge.net/wiki/index.php/Celera_Assembler_Terminology
[gfa]: https://github.com/pmelsted/GFA-spec/blob/master/GFA-spec.md
[paf]: https://github.com/lh3/miniasm/blob/master/PAF.md

## Getting Help

For detailed description of options, please see `man ./hifiasm.1`.
The `-h` option of hifiasm also provides simple description of options. If you
have further questions, please raise an issue at the issue page.

## Limitations and future works

1. For genome with low heterozygous rate, hifiasm only outputs
   haplotype-resolved assembly graph, instead of the phased chromosome-level
   assembly (will support such output in future).

2. For different species, hifiasm outputs different assembly graphs, which are not easy to use.
   Hifiasm will generate a universal haplotype-resolved contig graph for all species in future.

3. The running time and memory usage should be further reduced.

4. The N50 should be further improved. 
