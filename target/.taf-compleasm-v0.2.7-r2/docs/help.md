taf-compleasm 0.2.7-r2

TAFFISH wrapper for Compleasm, a genome and protein completeness assessment
tool based on BUSCO marker genes, miniprot, and HMMER.

Usage:
  taf-compleasm [TAF-APP-OPTION]
  taf-compleasm [COMMAND] [ARGS...]
  taf-compleasm -- [COMPLEASM-OPTION] [COMPLEASM-ARGS...]

TAF app options:
  -h, --help       Show this help text
  -v, --version    Show package and command version
  --compile        Print generated shell code instead of running it
  --               Stop parsing TAFFISH wrapper options

Upstream Compleasm:
  taf-compleasm compleasm --version
  taf-compleasm compleasm --help
  taf-compleasm compleasm download primates
  taf-compleasm compleasm run -a genome.fa -o compleasm-out -l eukaryota -t 8
  taf-compleasm compleasm run --autolineage -a genome.fa -o compleasm-out -t 8
  taf-compleasm compleasm protein -p proteins.faa -l eukaryota -o protein-out -t 8
  taf-compleasm compleasm list --local -L mb_downloads

Default-command option passthrough:
  taf-compleasm -- --version
  taf-compleasm -- --help

Bundled helper executables:
  taf-compleasm miniprot --version
  taf-compleasm hmmsearch -h
  taf-compleasm run_sepp.py -h
  taf-compleasm python -c 'import pandas, dendropy, compleasm'

Compleasm subcommands:
  run        Run miniprot alignment and completeness evaluation
  analyze    Evaluate completeness from a provided miniprot GFF
  download   Download BUSCO lineage datasets
  list       List local or remote BUSCO lineages
  miniprot   Run the Compleasm miniprot submodule
  protein    Evaluate completeness of protein sequences

Notes:
  - This command runs Compleasm inside the TAFFISH container image.
  - The upstream executable is named compleasm, so the clearest command-mode
    form is taf-compleasm compleasm ...
  - taf-compleasm --help and taf-compleasm --version are handled by the
    TAFFISH command wrapper. Use taf-compleasm compleasm --version or
    taf-compleasm -- --version for the upstream Compleasm version.
  - Do not use taf-compleasm run ... or taf-compleasm -- run ... for the
    Compleasm run subcommand. In TAFFISH command mode, the first non-option
    argument is treated as an executable inside the container.
  - taf-compleasm miniprot --version runs the standalone miniprot executable.
    taf-compleasm compleasm miniprot ... runs the Compleasm miniprot
    subcommand.
  - This image already includes pandas, miniprot, hmmsearch, and run_sepp.py;
    users do not need to install these inside the container.
  - Compleasm 0.2.7 uses BUSCO ODB12 by default and is not compatible with
    ODB10 lineage datasets.
  - download, list --remote, run with missing lineage data, and --autolineage
    may require network access and BUSCO lineage downloads.
  - The current image is linux/amd64 only. Docker and Podman runs embed
    --platform linux/amd64 in src/main.taf, so arm64 hosts can use amd64
    emulation without setting a global platform variable. This is not a native
    arm64 build; Apptainer compatibility depends on the host.

Container:
  image: ghcr.io/taffish/compleasm:0.2.7-r2
  supported backends: apptainer, podman, docker
  supported platforms: linux/amd64

Upstream:
  project: Compleasm
  repo:    https://github.com/huangnengCSU/compleasm
  release: https://github.com/huangnengCSU/compleasm/releases/tag/v0.2.7
  citation: Huang and Li 2023, doi:10.1093/bioinformatics/btad595
