FROM mambaorg/micromamba:1.5.10-bookworm-slim

ARG COMPLEASM_VERSION=0.2.9
ARG COMPLEASM_COMMIT=65414e3b207d40398cb9be5d58c9c0959200c9cf
ARG COMPLEASM_SOURCE_SHA256=3ebf11b3910cbdb4f503329b08d0ff080e03081068fe74eb3fdb7cbdda371d5d
ARG PYTHON_VERSION=3.11
ARG MINIPROT_VERSION=0.18
ARG HMMER_VERSION=3.4
ARG SEPP_VERSION=4.5.6
ARG DENDROPY_VERSION=5.0.8
ARG PANDAS_VERSION=2.2.3
ARG PASTA_VERSION=1.9.3
ARG PPLACER_VERSION=1.1.alpha22
ARG TARGETARCH

USER root

RUN set -eux; \
    for attempt in 1 2 3 4 5 6; do \
      if apt-get -o Acquire::Retries=3 update \
       && apt-get -o Acquire::Retries=3 install -y --no-install-recommends \
            bash \
            ca-certificates \
            curl \
            gzip \
            tar; then \
        break; \
      fi; \
      if [ "$attempt" = "6" ]; then exit 1; fi; \
      apt-get clean; \
      rm -rf /var/lib/apt/lists/*; \
      sleep 5; \
    done; \
    rm -rf /var/lib/apt/lists/*; \
    mkdir -p /tmp/build /opt/compleasm/share/doc/compleasm /opt/compleasm/share/licenses/compleasm; \
    chown -R mambauser:mambauser /tmp/build /opt/compleasm

USER mambauser
WORKDIR /tmp/build

ENV PYTHONDONTWRITEBYTECODE=1

RUN set -eux; \
    curl --retry 5 --retry-delay 2 --retry-all-errors -fsSL \
      "https://codeload.github.com/huangnengCSU/compleasm/tar.gz/${COMPLEASM_COMMIT}" \
      -o compleasm.tar.gz; \
    echo "${COMPLEASM_SOURCE_SHA256}  compleasm.tar.gz" | sha256sum -c -; \
    mkdir -p compleasm-src; \
    tar -xzf compleasm.tar.gz --strip-components=1 -C compleasm-src; \
    grep -Fx "__version__ = \"${COMPLEASM_VERSION}\"" compleasm-src/_version.py >/dev/null; \
    micromamba install -y -n base \
      -c conda-forge \
      -c bioconda \
      "python=${PYTHON_VERSION}" \
      pip \
      setuptools \
      wheel \
      "miniprot=${MINIPROT_VERSION}" \
      "hmmer=${HMMER_VERSION}" \
      "sepp=${SEPP_VERSION}" \
      "dendropy=${DENDROPY_VERSION}" \
      "pandas=${PANDAS_VERSION}" \
      "pasta=${PASTA_VERSION}" \
      "pplacer=${PPLACER_VERSION}"; \
    /opt/conda/bin/python -m pip install \
      --no-cache-dir \
      --no-deps \
      --no-build-isolation \
      ./compleasm-src; \
    cp compleasm-src/README.md /opt/compleasm/share/doc/compleasm/README.upstream.md; \
    cp compleasm-src/LICENSE /opt/compleasm/share/licenses/compleasm/LICENSE; \
    cp compleasm-src/LICENSE-BUSCO /opt/compleasm/share/licenses/compleasm/LICENSE-BUSCO; \
    micromamba list -n base --json > /opt/compleasm/share/doc/compleasm/conda-packages.json; \
    rm -rf /tmp/build/*; \
    micromamba clean --all --yes; \
    rm -rf /opt/conda/include /opt/conda/pkgs/*; \
    find /opt/conda -type d -name __pycache__ -prune -exec rm -rf '{}' +; \
    find /opt/conda -type f \( -name '*.a' -o -name '*.pyc' -o -name '*.pyo' \) -delete

ENV PATH=/opt/conda/bin:${PATH}
ENV PYTHONNOUSERSITE=1

USER root
WORKDIR /root

RUN set -eux; \
    { \
      printf "upstream_repo=https://github.com/huangnengCSU/compleasm\n"; \
      printf "upstream_tag=v%s\n" "${COMPLEASM_VERSION}"; \
      printf "upstream_commit=%s\n" "${COMPLEASM_COMMIT}"; \
      printf "source_sha256=%s\n" "${COMPLEASM_SOURCE_SHA256}"; \
      printf "target_arch=%s\n" "${TARGETARCH}"; \
    } > /opt/compleasm/share/doc/compleasm/SOURCE; \
    compleasm --version 2>&1 | grep -Fx "compleasm ${COMPLEASM_VERSION}" >/dev/null; \
    compleasm --help 2>&1 | grep -F "Compleasm modules help" >/dev/null; \
    compleasm run --help 2>&1 | grep -F -- "--odb" >/dev/null; \
    compleasm protein --help 2>&1 | grep -F "usage: compleasm protein" >/dev/null; \
    miniprot --version 2>&1 | grep -F "${MINIPROT_VERSION}" >/dev/null; \
    hmmsearch -h 2>&1 | grep -F "HMMER ${HMMER_VERSION}" >/dev/null; \
    run_sepp.py -h 2>&1 | grep -F "usage: run_sepp.py" >/dev/null; \
    command -v pplacer >/dev/null; \
    command -v guppy >/dev/null; \
    command -v java >/dev/null; \
    python -c 'from importlib.metadata import version; import compleasm, dendropy, pandas, sepp; assert version("compleasm") == "0.2.9"; assert version("sepp") == "4.5.6"; assert version("DendroPy") == "5.0.8"; assert version("pandas") == "2.2.3"'

RUN set -eux; \
    tmp=/tmp/taf-compleasm-build; \
    rm -rf "${tmp}"; \
    mkdir -p "${tmp}"; \
    printf "geneat1 type 2 100\n" > "${tmp}/odb10.lengths"; \
    printf "gene 100 5\n" > "${tmp}/odb12.2.lengths"; \
    python -c 'from compleasm import load_length_cutoff; a=load_length_cutoff("/tmp/taf-compleasm-build/odb10.lengths", "odb10"); b=load_length_cutoff("/tmp/taf-compleasm-build/odb12.2.lengths", "odb12.2"); assert a["geneat1"] == {"length": 100.0, "sigma": 2.0}; assert b["gene"] == {"length": 100.0, "sigma": 5.0}'; \
    printf ">chr1\nATGAAAACCGCTTACATTGCTAAACAACGTCAAATTTCTTTTGTTAAATCTCATTTTTCTCGTCAACTGGAAGAACGTCTGGGTCTGATTGAAGTTCAAGCTAATAAT\n" > "${tmp}/genome.fa"; \
    printf ">prot1\nMKTAYIAKQRQISFVKSHFSRQLEERLGLIEVQANN\n" > "${tmp}/proteins.faa"; \
    compleasm miniprot -a "${tmp}/genome.fa" -p "${tmp}/proteins.faa" -o "${tmp}/miniprot-out" -t 1 >/dev/null; \
    test -s "${tmp}/miniprot-out/miniprot_output.gff"; \
    test -e "${tmp}/miniprot-out/miniprot.done"; \
    rm -rf "${tmp}"

ENV TAFFISH_ENV=TAFFISH
ENV TAFFISH_NAME=compleasm
