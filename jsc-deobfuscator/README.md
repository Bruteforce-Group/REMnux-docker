Statically deobfuscate compiled V8 JavaScript bytecode (.jsc) protected with javascript-obfuscator, using [JSC Deobfuscator](https://github.com/hasherezade/jsc_deobfuscator) by Aleksandra "Hasherezade" Doniec. The image bundles the complete pipeline: the version-matched v8dasm disassembler, the View8 decompiler, brotli, and the deobfuscation filters, pinned to the upstream v1.0 release.

**Version lock:** the bundled v8dasm targets V8 10.2.154.26, the version used by JSCeal-era payloads. Payloads compiled with a different V8 build need a matching disassembler; see the [project wiki](https://github.com/hasherezade/jsc_deobfuscator/wiki/Building-V8-Disasm). The deobfuscation filters operate on View8 output and are not tied to a V8 version.

To run JSC Deobfuscator within this Docker container, create a directory where you'll store your .jsc payloads, e.g. ~/workdir. Then, use a command like this to launch the container and have your directory mapped as /home/nonroot/workdir inside the container:

```
docker run --rm -it -v ~/workdir:/home/nonroot/workdir remnux/jsc-deobfuscator
```

This image is amd64-only (v8dasm is a prebuilt x86-64 binary). On Apple Silicon and other arm64 hosts, add `--platform linux/amd64`.

The optional LLM-assisted function renaming (`deobf_ai.py`) needs an API key passed into the container, e.g. `-e ANTHROPIC_API_KEY` or `-e OPENAI_API_KEY`, or a reachable Ollama server.

The jsc-deobfuscator Docker image is hosted in the REMnux Docker Hub repository.

For documentation on JSC Deobfuscator, including the full pipeline walkthrough, see: https://github.com/hasherezade/jsc_deobfuscator/wiki
