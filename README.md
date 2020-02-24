# score-assemblies

A snakemake-wrapper for evaluating *de novo* bacterial genome assemblies, e.g. from Oxford Nanopore (ONT) or Illumina sequencing.

The workflow includes the following programs:
* [pomoxis](https://github.com/nanoporetech/pomoxis) assess_assembly and assess_homopolymers
* dnadiff from the [mummer](https://mummer4.github.io/index.html) package
* [QUAST](http://quast.sourceforge.net/quast)
* [BUSCO](https://busco.ezlab.org/)
* [ideel](https://github.com/mw55309/ideel/)

## Installation
Clone repository, for example:
```
git clone https://github.com/pmenzel/score-assemblies.git /opt/software/score-assemblies
```
Install dependencies into an isolated conda environment
```
conda env create -n score-assemblies --file /opt/software/score-assemblies/environment.yaml
```
and activate environment:
```
source activate score-assemblies
```

## Usage
First, prepare a data folder, which must contain subfolders for the assemblies
and the reference genomes, for example:
```
.
├── assemblies
│   ├── assemblies/flye-i4.fa
│   ├── assemblies/flye-i4+raconx4.fa
│   ├── assemblies/flye-i4+raconx4+medaka.fa
│   ├── assemblies/raven-p4.fa
│   └── assemblies/raven-p4+medaka.fa
│
└── references
    └── bac_sub_ref.fa
```

Run workflow in that folder, e.g. with 20 threads:
```
snakemake -k -s /opt/software/score-assemblies/Snakefile --cores 20
```

### Modules
score-assemblies will run these programs on each assembly:

#### assess_assembly and assess_homopolymers

Each assembly will be scored against each reference genome using the
`assess_assembly` and `assess_homopolymers` scripts from
[pomoxis](https://github.com/nanoporetech/pomoxis).  Additionally to the tables
and plots from pomoxis, summary plots for each reference genome will be plotted
in `<reference>_assess_assembly_all_meanQ.pdf` and
`<reference>_assess_homopolymers_all_correct_len.pdf`.

##### Example plot of assess_assembly results
![Example assess_assembly](example/example_assess_assembly.png?raw=true)

##### Example plot of assess_homopolymers results
![Example assess_homopolymers](example/example_assess_homopolymers.png?raw=true)

#### BUSCO

Set the lineage via the snakemake call:
```
snakemake -k -s /opt/software/score-assemblies/Snakefile --cores 20 --config busco_lineage=burkholderiales
```
If not set, the default lineage `bacteria` will be used.
Available datasets can be listed with `busco --list-datasets`

The number of complete, fragmented and missing BUSCOs per assembly is plotted in `busco/busco_stats.pdf`.

##### Example plot of BUSCO results
![Example BUSCO](example/example_assess_homopolymers.png?raw=true)

#### dnadiff
Each assembly is compared with each reference and the output files will be
located in `dnadiff/<reference>/<assembly>-dnadiff.report`.  The values for
`AvgIdentity` (from 1-to-1 alignments) and `TotalIndels` are extracted from these files and are plotted
for each reference in `dnadiff/<reference>_dnadiff_stats.pdf`.

##### Example plot of dnadiff results
![Example dnadiff](example/example_dnadiff.png?raw=true)

#### QUAST

One QUAST report is generated for each reference genome, containing the results for all assemblies.
The report files are located in `quast/<reference>/report.html`.

#### ideel

Open reading frames are predicted from each assembly via Prodigal and are
search in the Uniprot sprot database with diamond, retaining the best alignment
for each ORF. For each assembly, the distribution of the ratios between length
of the ORF and the matching database sequence are plotted to `ideel/ideel_stats.pdf`.

##### Example plot of ideel result
![Example ideel](example/example_ideel.png?raw=true)

