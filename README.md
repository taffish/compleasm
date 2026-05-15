# taf-compleasm

TAFFISH wrapper for [Compleasm](https://github.com/huangnengCSU/compleasm),
a fast genome and protein completeness assessment tool based on BUSCO marker
genes, miniprot, and HMMER.

This repository packages Compleasm 0.2.7 as a TAFFISH tool app. The container
installs the Bioconda Compleasm package and exposes the upstream `compleasm`
command together with its runtime tools, including `miniprot`, `hmmsearch`,
and `run_sepp.py`.

## Installation

Install from the public TAFFISH Hub index:

```sh
taf update
taf install compleasm
```

Install the exact release:

```sh
taf install compleasm 0.2.7-r1
```

For local testing before the app is published to the public index:

```sh
taf install --from .
```

## Usage

Show TAFFISH app help:

```sh
taf-compleasm --help
```

Show the TAFFISH package version:

```sh
taf-compleasm --version
```

Show the upstream Compleasm version:

```sh
taf-compleasm compleasm --version
taf-compleasm -- --version
```

Show upstream Compleasm help:

```sh
taf-compleasm compleasm --help
taf-compleasm -- --help
```

## Official Guide Mapping

The upstream quick-start commands use `compleasm_kit/compleasm.py`. In this
TAFFISH app, the equivalent upstream executable is available as `compleasm`
inside the container, so use `taf-compleasm compleasm ...`:

```sh
# upstream:
# compleasm_kit/compleasm.py download primates
taf-compleasm compleasm download primates

# upstream:
# compleasm_kit/compleasm.py run -t16 -l primates -a hg38.fa -o hg38-mb
taf-compleasm compleasm run -t 16 -l primates -a hg38.fa -o hg38-mb

# upstream autolineage mode, requiring SEPP:
# compleasm_kit/compleasm.py run --autolineage -a hg38.fa -o hs38-mb
taf-compleasm compleasm run --autolineage -a hg38.fa -o hs38-mb
```

The container already includes `pandas`, `miniprot`, `hmmsearch`, and
`run_sepp.py`, so users do not need to run `pip install pandas` or
`conda install sepp` inside the container.

Compleasm 0.2.7 uses BUSCO ODB12 by default and is not compatible with ODB10
lineage datasets. Use ODB12 lineage downloads unless you intentionally build a
separate older Compleasm release for ODB10.

Compleasm has subcommands such as `run`, `analyze`, `download`, `list`,
`miniprot`, and `protein`. Because this is a command-mode TAFFISH tool, the
clearest form is to name the upstream command explicitly:

```sh
taf-compleasm compleasm run -a genome.fa -o compleasm-out -l eukaryota -t 8
taf-compleasm compleasm protein -p proteins.faa -l eukaryota -o protein-out -t 8
taf-compleasm compleasm list --local -L mb_downloads
```

Do not use `taf-compleasm run ...` or `taf-compleasm -- run ...` for the
Compleasm `run` subcommand. In TAFFISH command mode, the first non-option
argument is treated as an in-container executable name. This means
`taf-compleasm miniprot --version` runs the standalone `miniprot` executable,
while `taf-compleasm compleasm miniprot ...` runs the Compleasm `miniprot`
subcommand.

Download a BUSCO lineage dataset:

```sh
taf-compleasm compleasm download eukaryota -L mb_downloads
```

Run with a specified lineage:

```sh
taf-compleasm compleasm run \
  -a genome.fa \
  -o compleasm-out \
  -l eukaryota \
  -L mb_downloads \
  -t 8
```

Run the miniprot-only submodule without downloading lineage data:

```sh
taf-compleasm compleasm miniprot -a genome.fa -p proteins.faa -o miniprot-out -t 8
```

Access bundled helper executables directly:

```sh
taf-compleasm miniprot --version
taf-compleasm hmmsearch -h
taf-compleasm run_sepp.py -h
```

## Package

```text
name: compleasm
command: taf-compleasm
version: 0.2.7-r1
kind: tool
image: ghcr.io/taffish/compleasm:0.2.7-r1
```

## Container

The container image is built from `docker/Dockerfile`. It starts from
`mambaorg/micromamba:1.5.10-bookworm-slim` and installs a pinned Bioconda
environment:

```text
compleasm 0.2.7
miniprot 0.18
hmmer 3.1b2
sepp 4.5.1
dendropy 4.5.2
pandas 1.3.5
python 3.7
```

The image includes these user-facing commands:

```text
compleasm
miniprot
hmmsearch
run_sepp.py
python
```

The current release is built for:

```text
linux/amd64
```

The official Compleasm release asset is x64 Linux, and the full Bioconda
environment for Compleasm 0.2.7 currently resolves cleanly on linux/amd64.
On linux/arm64, the available `sepp` packages require a newer DendroPy than
Compleasm 0.2.7 allows, so this TAFFISH release keeps full upstream
functionality and declares amd64 only.

On arm64 machines such as Apple Silicon Macs, the image can still be used
through Docker's amd64 emulation path. Request the amd64 platform and Docker
backend explicitly:

```sh
DOCKER_DEFAULT_PLATFORM=linux/amd64 TAFFISH_CONTAINER_BACKEND=docker \
  taf-compleasm compleasm --version

DOCKER_DEFAULT_PLATFORM=linux/amd64 TAFFISH_CONTAINER_BACKEND=docker \
  taf-compleasm compleasm run -a genome.fa -o compleasm-out -l eukaryota -t 8
```

This does not mean the image contains a native arm64 build; it runs the amd64
image through emulation. Podman or Apptainer on arm64 may require their own
amd64 compatibility setup or may reject the amd64-only image.

## Smoke Checks

The TAFFISH metadata declares a Docker smoke check:

```text
exist: compleasm, miniprot, hmmsearch, run_sepp.py, python
test:  Compleasm reports upstream version 0.2.7
test:  core Compleasm help and all six subcommand help pages are available
test:  miniprot, hmmsearch, and run_sepp.py are available
test:  Python can import pandas, dendropy, and compleasm
test:  a tiny local protein-to-genome miniprot workflow creates a GFF output
```

The smoke check covers all Compleasm command surfaces, but it does not download
BUSCO lineage datasets. Commands such as `download`, `list --remote`, `run`
with a missing lineage, and `--autolineage` can require network access and
external lineage data.

## Upstream

- Project: Compleasm
- Repository: <https://github.com/huangnengCSU/compleasm>
- Release: <https://github.com/huangnengCSU/compleasm/releases/tag/v0.2.7>
- Upstream license: Apache-2.0, with BUSCO-derived license terms noted by the
  upstream `LICENSE-BUSCO` file
- Primary citation: Huang and Li 2023, doi:10.1093/bioinformatics/btad595,
  PMID:37758247

## Maintainer Notes

Useful checks before publishing:

```sh
taf check
taf build
taf publish --release --dry-run
docker build --platform linux/amd64 -t ghcr.io/taffish/compleasm:0.2.7-r1 -f docker/Dockerfile .
docker run --rm ghcr.io/taffish/compleasm:0.2.7-r1 compleasm --version
docker run --rm ghcr.io/taffish/compleasm:0.2.7-r1 compleasm --help
```

The repository wrapper files are licensed under Apache-2.0. Upstream
Compleasm is distributed under Apache-2.0 with additional BUSCO-derived
license terms, and third-party runtime components are distributed under
their own upstream licenses.
