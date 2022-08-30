# Artifact for PacFuzz
This repository contains the data and artifact for PacFuzz.

```sh
- study-dataset.txt # dataset for the study
- tool/ # source code of PacFuzz
```

## Study dataset
The `study-dataset.txt` lists the CVEs we find in AFLGo, WindRanger, and Beacon for our empirical study presented in section III-B.
To reproduce our results, the interesting readers can just manually query the website of [MITRE CVE](https://cve.mitre.org/index.html) or [NIST NVD](https://cve.mitre.org/index.html) with the CVE identifiers.

## PacFuzz tool
### Installation
1. Joern for constructing control-dependency graph (CDG). Our modified copy of Joern is at `tool/joern`.
- Follow the instructions for building form source code: https://docs.joern.io/installation.
- It requires [scala build tool (sbt)](https://www.scala-sbt.org/download.html) for the installation.

2. AFL for the fuzzing loop. The associated code is at `tool/afl`.
- Follow the instructions of AFLGo to build the fuzzer. In particular, run `make` in `tool/afl/` and `tool/afl/llvm-mode/`, respectively.

## Use
The package include code for running PacFuzz for crash reproduction.
- Parse target state

- Compute basic block distances

- Instrument

- Fuzz


