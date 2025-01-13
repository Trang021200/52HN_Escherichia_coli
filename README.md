# 52HN - Evolutionary dynamics of _Escherichia coli_ in Vietnam
**Bioinformatics workflow of _E. coli_ whole-genome data, collected as part of the 52HN project**

# Different steps

## Species identification 
Species identification was conducted using Mash 

[https://mash.readthedocs.io/en/latest/](https://mash.readthedocs.io/en/latest/)

Used database for screening: [RefSeq88n.msh.gz](https://obj.umiacs.umd.edu/mash/screen/RefSeq88n.msh.gz)

```
% mash screen  -p [threads] RefSeq88n.msh.gz [sample] > mash_output.tab
# Extracting the top 10 hits 
% sort -gr mash_output.tab -o mash_output_sort.tab
% head -n 10 mash_output_sor.tab > mash_output_top10.tab
```

Three genomes were identified as non _E.coli_: 17.6a, 17.6b, 17.6c

:heavy_check_mark: Mash: v2.3

## Quality control
Quality control was done by [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/)
[Fastp](https://github.com/OpenGene/fastp) was used for filtering reads with more than 30% of bases with Phred score below 15 and/or reads with more than 5 N-bases. Adapters trimming is disabled.

```
% fastp -i [input.read1] -I [input.read2] -o [output.fastp1] -O [output.fastp2] -q 15 -u 30 -n 5 -A
```
:heavy_check_mark: FastQC: 0.11.5
:heavy_check_mark: Fastp: 0.24.0

## Assembly 
The assembly was done with [SPAdes v4.0.0](https://ablab.github.io/spades/) using the k-mers: 21, 35, 55, 71, 91, 111 

[Page AJ, De Silva N, Hunt M, Quail MA, Parkhill J, Harris SR, Otto TD, Keane JA. Robust high-throughput prokaryote de novo assembly and improvement pipeline for Illumina data. Microb Genom. 2016 Aug 25;2(8):e000083. doi: 10.1099/mgen.0.000083. PMID: 28348874; PMCID: PMC5320598.](https://doi.org/10.1099/mgen.0.000083)
```
% spades.py --isolate -1 [read1] -2 [read2] -o [output] --threads [threads] -k 21,35,55,71,91,111 --cov-cutoff auto

--isolate: This flag is highly recommended for high-coverage isolate and multi-cell Illumina data; improves the assembly quality and running time. We also recommend trimming your reads prior to the assembly. This option is not compatible with --only-error-correction or --careful options.

--cov-cutoff: Read coverage cutoff value. Must be a positive float value, or "auto", or "off". Default value is "off". When set to "auto" SPAdes automatically computes coverage threshold using conservative strategy.
```
[More details on --cov-cutoff parameter (this should be handled with care](https://github.com/ablab/spades/issues/18)

:heavy_check_mark: SPAdes: 4.0.0

## Quality control of the assembly 
1. Assemblies' qualities were checked with [QUAST](https://github.com/ablab/quast) using default parameter
2. Assemblies' completeness were assessed using [BUSCO](https://busco.ezlab.org/) against database enterobacteriaceae_odb12
```
# Batch mode
% busco -i [inputs.directory] -m geno -l enterobacteriaceae_odb12 -c [threads] 
```
3. Assembly filtration: contigs with coverage lower than 1 were discarded using [SeqKit](https://bioinf.shenwei.me/seqkit/)
```
% seqkit fx2tab [assembly] | csvtk mutate -H -t -f 1 -p "cov_(.+)" | awk -F "\t" '$4>=1' | seqkit tab2fx > filtered_assembly.fasta
```
:heavy_check_mark: BUSCO: 5.8.2
:heavy_check_mark: seqkit: v2.8.2

## Serotype 
Serotypes were determined using [ECTyper](https://github.com/phac-nml/ecoli_serotyping) with default parameters.

:heavy_check_mark: ECTyper: 2.0.0

## MLST
Sequence Typing was done using [mlst](https://github.com/tseemann/mlst), --scheme ecoli_achtman_4, and default parameters.

:heavy_check_mark: mlst 

## Phylogroup 
The phylogroup was determined using [ClermonTyping](https://github.com/A-BN/ClermonTyping) and default parameters.

:heavy_check_mark: ClermonTyping: 24.02

## Virulence genes typing
The virulenece genes were screen against the curated database "Septicoli" using [Abricate](https://github.com/tseemann/abricate) with thresholds --minid 80 and --mincov 90. The database was done by Guilhem Royer in Kieffer et al, 2019. 

[Kieffer N, Royer G, Decousser JW, Bourrel AS, Palmieri M, Ortiz De La Rosa JM, Jacquier H, Denamur E, Nordmann P, Poirel L. **mcr-9, an Inducible Gene Encoding an Acquired Phosphoethanolamine Transferase in Escherichia coli, and Its Origin**. Antimicrob Agents Chemother. 2019 Aug 23;63(9):e00965-19. doi: 10.1128/AAC.00965-19. Erratum in: Antimicrob Agents Chemother. 2019 Oct 22;63(11): PMID: 31209009; PMCID: PMC6709461. https://doi.org/10.1128/AAC.00965-19](https://doi.org/10.1128/AAC.00965-19)

:heavy_check_mark: abricate: 1.0.1

## Resistance genes typing 
The resistance genes were screened against the NCBI database using [AMRfinderplus](https://github.com/ncbi/amr) 

```
% amrfinder -n [assembly] --organism Escherichia --threads [threads] --plus --name [prefix] > output.tab
```

:heavy_check_mark: amrfinder: 4.0.3

## Plasmid 
Plasmid replicon types were screened against the PlasmidFinder_DB database using [Abricate](https://github.com/tseemann/abricate) with thresholds --minid 80 and --mincov 90. 

```
% abricate --fofn [file of file names] --db plasmidfinder > output.tab
```

:heavy_check_mark: abricate: 1.0.1

## Annotation 
The annotation was done using [Bakta](https://github.com/oschwengers/bakta)

```
#Bakta database needs to be downloaded manually per installation instruction and kept in bakta_db directory 
% bakta --db [bakta_db] --prefix [file_name] --output [out_dir] --keep-contig-headers --genus Escherichia -v [assembly] -t [threads]
```
:heavy_check_mark: bakta: 1.10.3

## Pan-genome investigation
Pangenome calculation was done using [Panaroo](https://github.com/gtonkinhill/panaroo)

```
% panaroo -i [input_gff3_files] -o [out_dir] --clean-mode strict --remove-invalid-genes -a core

--clean-mode: The stringency mode at which to run panaroo. Must be one of 'strict','moderate' or 'sensitive'. Each of these modes can be fine tuned using the additional parameters in the 'Graph correction' section. strict: Requires fairly strong evidence (present in  at least 5% of genomes) to keep likely contaminant genes. Will remove genes that are refound more often than they were called originally.
--remove-invalid-genes: removes annotations that do not conform to the expected Prokka format such as those including premature stop codons.
-a: Output alignments of core genes or all genes. Options are 'core' and 'pan'. Default: 'None'
#Aligner: default 'mafft'
```
:heavy_check_mark: Panaroo: 1.5.1

##Phylogenomic inference
The phylogenetic tree was constructed using IQTree 

```
#Due to large number of samples and limited computing capacity, the model GTR+F+R7 was chosen in prior instead of running model search by ModelFinder (-m MFP). The limits for memory and cores usage were 54G (-mem) and 18 (-ntmax), respectively. Safe numerical mode (-safe) was turned on to avoid numerical underflow for large data sets with many sequences (this mode is automatically turned on when having more than 2000 sequences)
% iqtree -s [msa.aln] -pre [file_prefix] -m GTR+F+R7 -bb 1000 -alrt 1000 -nt AUTO -mem 54G -ntmax 18 -safe

-bb: number of bootstrap replicates
-alrt: number of replicates to perform SH-like
```
:heavy_check_mark: IQTree: 1.6.12

## Results

All the typing results were processed using R 4.3.3.












