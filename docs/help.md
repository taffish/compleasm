taf-compleasm 0.2.9-r1

Purpose:
  Run Compleasm genome or protein completeness assessment with BUSCO marker
  genes. The image includes miniprot, HMMER, SEPP, PASTA, pplacer, and Java,
  but it does not include BUSCO lineage datasets.

Usage:
  taf-compleasm -- --help
  taf-compleasm -- --version
  taf-compleasm compleasm SUBCOMMAND [ARGS...]
  taf-compleasm COMMAND [ARGS...]

Common workflows:
  taf-compleasm compleasm download eukaryota -L mb_downloads --odb odb12
  taf-compleasm compleasm run -a genome.fa -o out -l eukaryota \
    -L mb_downloads --odb odb12 -t 8
  taf-compleasm compleasm protein -p proteins.faa -o out -l eukaryota \
    -L mb_downloads --odb odb12 -t 8
  taf-compleasm compleasm miniprot -a genome.fa -p proteins.faa -o out -t 8

Compleasm subcommands:
  run        Align assembly sequences and assess completeness
  analyze    Assess completeness from an existing miniprot GFF
  download   Download a BUSCO lineage
  list       List local or remote lineages
  miniprot   Run the Compleasm miniprot-only stage
  protein    Assess protein-sequence completeness

Packaged commands:
  compleasm      Upstream Compleasm CLI
  miniprot       Protein-to-genome aligner
  hmmsearch/hmmbuild/hmmalign  HMMER profile tools
  run_sepp.py    SEPP placement entry point
  pplacer/guppy  SEPP placement helpers
  java/python    PASTA/SEPP and Compleasm runtimes

Command mode:
  The default command is compleasm. Use -- before option-leading arguments:
    taf-compleasm -- --help
    taf-compleasm -- --version

  Name compleasm explicitly before an upstream subcommand:
    taf-compleasm compleasm run ...

  Do not use taf-compleasm run ... because command mode would look for an
  executable named run. taf-compleasm miniprot ... invokes the standalone
  miniprot executable; taf-compleasm compleasm miniprot ... invokes the
  Compleasm subcommand.

Inputs:
  assembly FASTA      Genome or transcript assembly
  protein FASTA       Predicted protein sequences
  miniprot GFF        Existing miniprot alignment for analyze
  lineage directory  External BUSCO data selected with -l, -L, and --odb

Key outputs:
  result directory      Upstream summary tables and intermediate results
  miniprot_output.gff   Alignment output from the miniprot submodule

Databases and network:
  BUSCO lineage data is not bundled. Compleasm 0.2.9 accepts ODB10, ODB12, and
  ODB12.2 formats; ODB12 is the default. The lineage must match --odb.

  download and list --remote need network access. run may download a missing
  lineage, and --autolineage may also need remote data. Download into a
  persistent host directory with -L, then reuse that directory offline:
    taf-compleasm compleasm list --local -L mb_downloads --odb odb12

Platform and resources:
  image: ghcr.io/taffish/compleasm:0.2.9-r1
  platforms: linux/amd64, linux/arm64
  backends: apptainer, podman, docker
  Production cost depends on assembly size, lineage size, threads, and
  autolineage. The package smoke is not a production resource benchmark.

Boundaries:
  The image includes the standard and autolineage executables, but no lineage
  data. Offline smoke does not download data, run a full lineage assessment,
  or perform a positive autolineage analysis.

Detailed documentation:
  https://github.com/taffish/compleasm
  https://github.com/huangnengCSU/compleasm

Wrapper options:
  taf-compleasm --help       Show this TAFFISH help.
  taf-compleasm --version    Show TAFFISH wrapper version.
  taf-compleasm --compile    Compile the TAFFISH wrapper.
  taf-compleasm -- --help    Pass --help to the default compleasm command.
