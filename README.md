# compleasm

`compleasm` packages [Compleasm](https://github.com/huangnengCSU/compleasm)
for TAFFISH.

Package identity:

- name: `compleasm`
- command: `taf-compleasm`
- kind: `tool`
- version: `0.2.9-r1`
- image: `ghcr.io/taffish/compleasm:0.2.9-r1`
- upstream release: `v0.2.9`
- runtime version: `compleasm 0.2.9`
- license: Apache-2.0
- upstream: <https://github.com/huangnengCSU/compleasm>

## What This App Packages

Compleasm performs genome or protein completeness assessment with BUSCO marker
genes. The app builds upstream Compleasm 0.2.9 from its exact release commit and
ships the runtime tools required by its standard and autolineage paths,
including miniprot, HMMER, SEPP, PASTA, pplacer, and Java.

Compleasm 0.2.9 supports ODB10, ODB12, and ODB12.2 lineage formats. ODB12 is
the upstream default.

## Scope

This app supports:

- assembly completeness with `compleasm run`
- protein completeness with `compleasm protein`
- analysis of an existing miniprot GFF with `compleasm analyze`
- lineage download, local/remote listing, and autolineage
- standalone access to packaged helper executables

This app does not:

- bundle BUSCO lineage datasets
- make remote downloads reproducible or available while offline
- replace biological review of lineage choice and completeness results

## Container Contents

- `compleasm`: upstream command and all six public subcommands
- `miniprot`: protein-to-genome aligner used by assembly mode
- `hmmsearch`, `hmmbuild`, `hmmalign`: HMMER programs
- `run_sepp.py`, `pplacer`, `guppy`: autolineage placement stack
- `python`: pinned Python runtime with Compleasm, pandas, DendroPy, and SEPP

The source archive commit and checksum, upstream licenses, upstream README, and
resolved Conda package inventory are retained under
`/opt/compleasm/share/`.

## Usage

Install this exact release:

```sh
taf install compleasm 0.2.9-r1
```

Show upstream help and version:

```sh
taf-compleasm -- --help
taf-compleasm -- --version
```

Download a lineage into a host-visible directory:

```sh
taf-compleasm compleasm download eukaryota -L mb_downloads --odb odb12
```

Run assembly completeness:

```sh
taf-compleasm compleasm run \
  -a genome.fa \
  -o compleasm-out \
  -l eukaryota \
  -L mb_downloads \
  --odb odb12 \
  -t 8
```

Run protein completeness:

```sh
taf-compleasm compleasm protein \
  -p proteins.faa \
  -o protein-out \
  -l eukaryota \
  -L mb_downloads \
  --odb odb12 \
  -t 8
```

Run the miniprot-only submodule without lineage data:

```sh
taf-compleasm compleasm miniprot \
  -a genome.fa \
  -p proteins.faa \
  -o miniprot-out \
  -t 8
```

## Command Mode

The default in-container command is `compleasm`. Option-leading arguments can
be passed with `--`:

```sh
taf-compleasm -- --help
taf-compleasm -- --version
```

For a Compleasm subcommand, name the upstream executable explicitly:

```sh
taf-compleasm compleasm run ...
taf-compleasm compleasm miniprot ...
```

Do not use `taf-compleasm run ...`: automatic command mode would look for an
executable named `run`. Conversely, `taf-compleasm miniprot --version` invokes
the standalone miniprot executable, not the Compleasm subcommand.

## Inputs

| Input | Meaning | Notes |
| --- | --- | --- |
| FASTA assembly | Genome or transcript assembly | Used by `run`, `analyze`, or `miniprot` as appropriate |
| Protein FASTA | Predicted protein sequences | Used by `protein` or `miniprot` |
| miniprot GFF | Existing miniprot alignment | Used by `analyze` |
| BUSCO lineage directory | Marker profiles and metadata | Download separately and select with `-l`, `-L`, and `--odb` |

Input and output paths should be under the working directory or another path
made visible by TAFFISH runtime configuration.

## Output Notes

`run`, `analyze`, and `protein` create an upstream Compleasm output directory
containing summary tables and intermediate results. `miniprot` writes
`miniprot_output.gff` and a completion marker in its output directory. Exact
filenames and interpretation follow upstream Compleasm.

Compleasm rejects an already populated output directory in several modes.
Choose a new path or manage existing results deliberately.

## Resources, Databases, and Platform

Native images are built for `linux/amd64` and `linux/arm64`.

Lineage data is external. `download` and `list --remote` require network
access; `run` may download a missing lineage, and `--autolineage` may also need
remote lineage data. Download once into a persistent host directory and reuse
it with `-L` for offline runs:

```sh
taf-compleasm compleasm list --local -L mb_downloads --odb odb12
```

ODB10, ODB12, and ODB12.2 datasets are accepted by this release, but a lineage
must match the selected `--odb` value. Lineage availability and contents are
controlled by BUSCO/Compleasm upstream and are not frozen in this image.

Runtime cost depends on assembly size, lineage size, thread count, and whether
autolineage is enabled. Production genomes need substantially more time and
memory than the tiny package smoke.

## Boundaries

The image includes all packaged executables needed for upstream standard and
autolineage paths, but not external lineage datasets. Network-dependent
commands remain subject to upstream service availability. Passing custom
lineage data or non-default helper options is supported according to upstream
CLI behavior, not a separate TAFFISH API.

## Troubleshooting

- If a lineage is not found, verify `-L`, `-l`, and `--odb` together, then run
  `compleasm list --local` against the same directory.
- If `taf-compleasm run ...` reports a missing executable, use
  `taf-compleasm compleasm run ...`.
- If a remote download fails, retry outside an offline environment or prepare
  the lineage directory separately and reuse it locally.

## Testing

The smoke test independently checks:

- exact upstream source identity and Python package versions
- every Compleasm subcommand help surface
- miniprot, HMMER, SEPP, pplacer, and guppy availability, plus execution of
  the Java-based SEPP JSON merger
- ODB10 and ODB12.2 cutoff parsing plus local lineage discovery
- a tiny real HMMER search and a tiny real Compleasm miniprot output

It does not download a lineage, run a full completeness assessment, or perform
a positive autolineage analysis. Those paths require substantial external data
and do not belong in an offline index smoke.

## License and Citation

TAFFISH app packaging is Apache-2.0. Upstream Compleasm is Apache-2.0 and also
ships `LICENSE-BUSCO`; third-party runtime components retain their own terms.

Cite Compleasm as described upstream: Huang and Li (2023),
<https://doi.org/10.1093/bioinformatics/btad595>.
