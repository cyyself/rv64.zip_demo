# rv64.zip demo

This is a demo repo for the rv64.zip project.

## Pre-requisites

To reproduce the results, you need the following hardware:

- A RISC-V Hardware with RV64GCBV_Zicond ISA support and Zicntr + Zihpm support to run perf record and has 8 cores.
  - For hardware without 8 cores, you can just delete "numactl --physcpubind=7" in `coremark/Makefile`
  - Spacemit K1 / M1 Recommanded.
  - We have set `r22:u` for cycles and `r4d:u` for instructions in the `coremark/Makefile` for the Spacemit K1 / M1 hardware, for other hardware, you can use `perf record -e cycles:u -e instructions:u` to get the perf data if its supported, and also change other parts in the `coremark/Makefile` accordingly.
- An ssh target called "musepi" that points to the RISC-V hardware.
  - For local development, you can delete the "ssh musepi" in `coremark/Makefile`
- For cross compile, you should make sure this folder is shared between the host and the target through NFS.

For software, we need:
- Linux-perf
  - You may need to set `sudo sysctl -w kernel.perf_event_paranoid=-1` on your target to allow perf to access the PMU.
- Python3
- The dependencies to build the GCC toolchain

## Build

### GCC

- For RISC-V Native GCC:

```bash
cd gcc
mkdir build
cd build
../configure --prefix=$(pwd)/install --enable-languages=c,c++ --disable-multilib --disable-bootstrap --disable-libsanitizer
make -j8 && make install
alias riscv64-unknown-linux-gnu-gcc=$(pwd)/install/bin/gcc
```

- For Cross-Compile GCC:

We recommand you to follow [riscv-gnu-toolchain](https://github.com/riscv-collab/riscv-gnu-toolchain) and replace the `gcc` submodule with the one in this repo.

And we should finally get `riscv64-unknown-linux-gnu-gcc` command in the `PATH`.


### CoreMark

Note: Our profiler needs to access CPU cycles, so please don't use `-j $THREADS` to build the coremark benchmark. Otherwise, the perf data will not be precise.


### Get target profile

```bash
cd coremark
make target_profile.txt
```

You will get a file called `target_profile.txt` in the `coremark` folder, which contains the target profile for the coremark benchmark:

```
matrix_test:default#arch=+zba,+zbb
crcu32:default#arch=+zba,+zbb
core_bench_state:default#arch=+zba,+zbb#arch=+b,+v,+zicond
```

### Benchmark all the results

```bash
cd coremark
make all_result
```

You will get the summarized results like this:

```
cat result-rv64gc.txt | grep "Iterations/Sec"
Iterations/Sec   : 5576.962077
cat result-rv64gc_zba_zbb.txt | grep "Iterations/Sec"
Iterations/Sec   : 5996.511121
cat result-rv64gcbv.txt | grep "Iterations/Sec"
Iterations/Sec   : 5744.725298
cat result-rv64gcbv_zicond.txt | grep "Iterations/Sec"
Iterations/Sec   : 5719.633943
cat result-coremark-autofmv.txt | grep "Iterations/Sec"
Iterations/Sec   : 5971.445633
```