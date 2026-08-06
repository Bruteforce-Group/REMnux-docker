# Name: JSC Deobfuscator
# Website: https://github.com/hasherezade/jsc_deobfuscator
# Description: Statically deobfuscate compiled V8 JavaScript bytecode (.jsc)
#   that was protected with javascript-obfuscator, using View8 pseudocode as
#   the working representation. Developed and tested on JSCeal payloads.
# Category: Statically Analyze Code: Scripts
# Author: Aleksandra "Hasherezade" Doniec @ Check Point Research
# License: GPL-2.0-or-later: https://github.com/hasherezade/jsc_deobfuscator/blob/main/LICENSE
# Notes: jsc-deobfuscator
#
# IMPORTANT: The bundled v8dasm disassembler targets V8 10.2.154.26, the
# version used by JSCeal-era payloads. Payloads compiled with a different V8
# build need a matching v8dasm; see the project wiki for rebuild instructions:
# https://github.com/hasherezade/jsc_deobfuscator/wiki/Building-V8-Disasm
# The deobfuscation filters themselves operate on View8 output and are not
# tied to a V8 version.
#
# This image is amd64-only (v8dasm is a prebuilt x86-64 binary). On Apple
# Silicon and other arm64 hosts, add: --platform linux/amd64
#
# To run JSC Deobfuscator within this Docker container, create a directory
# where you'll store your .jsc payloads, e.g. ~/workdir. Then, use a command
# like this to launch the container and have your directory mapped as
# /home/nonroot/workdir inside the container:
#
#   docker run --rm -it -v ~/workdir:/home/nonroot/workdir remnux/jsc-deobfuscator
#
# Inside the container, the pipeline for a Brotli-compressed payload is:
#
#   brotli -d app.jsc -o app.decompressed.jsc
#   v8dasm app.decompressed.jsc > app.jsc.disasm.txt
#   mkdir -p decompiled deobfuscated
#   python3 $JSC_DEOBF_ROOT/View8/view8.py --input_format disassembled \
#     --inp app.jsc.disasm.txt --normalize \
#     --out decompiled/app.dec.txt --export_format decompiled serialized
#   python3 $JSC_DEOBF_ROOT/deobf_all.py --inp decompiled/app.dec.pkl \
#     --out deobfuscated/app.deobf.txt --export_format decompiled serialized
#
# The batch helper scripts also work from the current directory, e.g.:
#
#   bash $JSC_DEOBF_ROOT/scripts/unpack_all.sh
#
# The optional LLM-assisted function renaming (deobf_ai.py) needs an API key
# passed into the container, e.g. -e ANTHROPIC_API_KEY or -e OPENAI_API_KEY,
# or a reachable Ollama server.
#
# The jsc-deobfuscator Docker image is hosted in the REMnux Docker Hub
# repository.
#
# For documentation on JSC Deobfuscator, see:
# https://github.com/hasherezade/jsc_deobfuscator/wiki

# slim-trixie (not bookworm): the prebuilt v8dasm needs GLIBC_2.38 and
# GLIBCXX_3.4.32, newer than Debian 12 provides
FROM python:3.12-slim-trixie
LABEL maintainer="Lenny Zeltser (@lennyzeltser, www.zeltser.com)"
LABEL updated="6 Aug 2026"
LABEL updated_by="lennyzeltser"

ENV LANG=C.UTF-8
ENV LANGUAGE=C.UTF-8
ENV LC_ALL=C.UTF-8

USER root

# Install deps, clone the pinned release with the View8 submodule, fetch the
# prebuilt v8dasm release asset and verify its published SHA256, then remove
# build-only packages in a single layer to avoid shipping dead weight
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    git \
    curl \
    ca-certificates \
    brotli \
    libstdc++6 && \
    git clone --recurse-submodules --shallow-submodules --depth 1 \
      --branch v1.0 \
      https://github.com/hasherezade/jsc_deobfuscator.git /opt/jsc_deobfuscator && \
    pip3 install --no-cache-dir \
      -r /opt/jsc_deobfuscator/requirements.txt \
      openai && \
    if [ -f /opt/jsc_deobfuscator/View8/requirements.txt ]; then \
      pip3 install --no-cache-dir -r /opt/jsc_deobfuscator/View8/requirements.txt; \
    fi && \
    curl -fsSL -o /usr/local/bin/v8dasm \
      https://github.com/hasherezade/jsc_deobfuscator/releases/download/v1.0/v8dasm && \
    echo "32be6bae25915c2736b131e6f529d8341c2f6d31a93a8f0b83e4db62c8ba9c7b  /usr/local/bin/v8dasm" | sha256sum -c - && \
    chmod 755 /usr/local/bin/v8dasm && \
    apt-get purge -y git curl && \
    apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/*

# The helper scripts read these; every path in scripts/config.sh can be
# overridden through the environment
ENV JSC_DEOBF_ROOT=/opt/jsc_deobfuscator
ENV V8DASM=/usr/local/bin/v8dasm

RUN groupadd -r nonroot && \
    useradd -m -r -g nonroot -d /home/nonroot -s /usr/sbin/nologin -c "Nonroot User" nonroot && \
    mkdir -p /home/nonroot/workdir && \
    chown -R nonroot:nonroot /home/nonroot

USER nonroot
ENV HOME=/home/nonroot
ENV USER=nonroot
WORKDIR /home/nonroot/workdir
VOLUME ["/home/nonroot/workdir"]
CMD ["/bin/bash"]
