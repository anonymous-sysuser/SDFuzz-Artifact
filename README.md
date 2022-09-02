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
```sh
cd tool/afl/scripts/
./parseTrace.py -t TRACE -o TRACE.OUT
```
- Compute basic block distances
```sh
# Set aflgo-instrumenter
export CC=$AFLGO/afl-clang-fast
export CXX=$AFLGO/afl-clang-fast++

# Set aflgo-instrumentation flags
export COPY_CFLAGS=$CFLAGS
export COPY_CXXFLAGS=$CXXFLAGS
export ADDITIONAL="-targets=$TMP_DIR/BBtargets.txt -outdir=$TMP_DIR -flto -fuse-ld=gold -Wl,-plugin-opt=save-temps"
export CFLAGS="$CFLAGS $ADDITIONAL"
export CXXFLAGS="$CXXFLAGS $ADDITIONAL"

# Build libxml2 (in order to generate CG and CFGs).
# Meanwhile go have a coffee ☕️
export LDFLAGS=-lpthread
pushd $SUBJECT
  ./autogen.sh
  ./configure --disable-shared
  make clean
  make xmllint
popd
# * If the linker (CCLD) complains that you should run ranlib, make
#   sure that libLTO.so and LLVMgold.so (from building LLVM with Gold)
#   can be found in /usr/lib/bfd-plugins
# * If the compiler crashes, there is some problem with LLVM not 
#   supporting our instrumentation (afl-llvm-pass.so.cc:540-577).
#   LLVM has changed the instrumentation-API very often :(
#   -> Check LLVM-version, fix problem, and prepare pull request.
# * You can speed up the compilation with a parallel build. However,
#   this may impact which BBs are identified as targets. 
#   See https://github.com/aflgo/aflgo/issues/41.


# Test whether CG/CFG extraction was successful
$SUBJECT/xmllint --valid --recover $SUBJECT/test/dtd3
ls $TMP_DIR/dot-files
echo "Function targets"
cat $TMP_DIR/Ftargets.txt

# Clean up
cat $TMP_DIR/BBnames.txt | rev | cut -d: -f2- | rev | sort | uniq > $TMP_DIR/BBnames2.txt && mv $TMP_DIR/BBnames2.txt $TMP_DIR/BBnames.txt
cat $TMP_DIR/BBcalls.txt | sort | uniq > $TMP_DIR/BBcalls2.txt && mv $TMP_DIR/BBcalls2.txt $TMP_DIR/BBcalls.txt

$AFLGO/script/parseTrace.py -t TRACE -o $TMP_DIR/targets.txt
cd tool/joern/
./joern-parse PROGRAM #by default generates ./cpg.bin
./joern-export --repr cdg --out CDG
./joern-export --repr cdg --out AST

# Generate distance ☕️
# $AFLGO/scripts/genDistance.sh is the original, but significantly slower, version
$AFLGO/scripts/genDistance.sh $SUBJECT $TMP_DIR xmllint

# Check distance file
echo "Distance values:"
head -n5 $TMP_DIR/distance.cfg.txt
echo "..."
tail -n5 $TMP_DIR/distance.cfg.txt
```
- Instrument
Compile the program again but not in the preprocessing mode.

- Fuzz it like AFL/AFLGo.



