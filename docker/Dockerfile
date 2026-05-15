FROM mambaorg/micromamba:1.5.10-bookworm-slim

ARG COMPLEASM_VERSION=0.2.7
ARG MINIPROT_VERSION=0.18
ARG HMMER_VERSION=3.1b2
ARG SEPP_VERSION=4.5.1
ARG DENDROPY_VERSION=4.5.2
ARG PANDAS_VERSION=1.3.5

USER root

RUN apt-get update \
 && apt-get install -y --no-install-recommends ca-certificates bash \
 && rm -rf /var/lib/apt/lists/*

USER mambauser

RUN micromamba install -y -n base \
      -c conda-forge \
      -c bioconda \
      "python=3.7" \
      "compleasm=${COMPLEASM_VERSION}" \
      "miniprot=${MINIPROT_VERSION}" \
      "hmmer=${HMMER_VERSION}" \
      "sepp=${SEPP_VERSION}" \
      "dendropy=${DENDROPY_VERSION}" \
      "pandas=${PANDAS_VERSION}" \
 && micromamba clean -a -y

ENV PATH=/opt/conda/bin:${PATH}

USER root

RUN set -eux; \
    compleasm --version | grep -F "compleasm ${COMPLEASM_VERSION}" >/dev/null; \
    compleasm --help | grep -F "Compleasm modules help" >/dev/null; \
    compleasm run --help | grep -F "usage: compleasm run" >/dev/null; \
    compleasm analyze --help | grep -F "usage: compleasm analyze" >/dev/null; \
    compleasm download --help | grep -F "usage: compleasm download" >/dev/null; \
    compleasm list --help | grep -F "usage: compleasm list" >/dev/null; \
    compleasm miniprot --help | grep -F "usage: compleasm miniprot" >/dev/null; \
    compleasm protein --help | grep -F "usage: compleasm protein" >/dev/null; \
    miniprot --version 2>&1 | grep -F "${MINIPROT_VERSION}" >/dev/null; \
    hmmsearch -h | grep -F "HMMER" >/dev/null; \
    run_sepp.py -h | grep -F "usage: run_sepp.py" >/dev/null; \
    python -c "import pandas, dendropy; import compleasm"

RUN set -eux; \
    mkdir -p /tmp/taf-compleasm-test; \
    cd /tmp/taf-compleasm-test; \
    printf ">chr1\nATGAAAACCGCTTACATTGCTAAACAACGTCAAATTTCTTTTGTTAAATCTCATTTTTCTCGTCAACTGGAAGAACGTCTGGGTCTGATTGAAGTTCAAGCTAATAAT\n" > genome.fa; \
    printf ">prot1\nMKTAYIAKQRQISFVKSHFSRQLEERLGLIEVQANN\n" > proteins.faa; \
    compleasm miniprot -a genome.fa -p proteins.faa -o miniprot-out -t 1 >/dev/null; \
    test -s miniprot-out/miniprot_output.gff; \
    test -e miniprot-out/miniprot.done; \
    cd /; \
    rm -rf /tmp/taf-compleasm-test

WORKDIR /root

ENV TAFFISH_ENV=TAFFISH
ENV TAFFISH_NAME=compleasm
