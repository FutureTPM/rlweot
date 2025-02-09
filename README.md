# Lattice-Based Random Oblivious Transfer

This repository and source code are the artifact for the paper
"ROTed: Random Oblivious Transfer for embedded devices" published at
[TCHES 2021](https://tches.iacr.org/index.php/TCHES/article/view/9065).
As such, we strongly recommend reading the paper before using
the software herein.

The repository contains multiple implementations for random and non-random
oblivious transfer (OT) algorithms. All code should build on ARM and x86
machines.

## Dependencies

In order to run the software the user needs to install multiple
dependencies. The names of the dependencies will vary by Linux distribution and
operating system. Some dependencies are optional while others are required for
the code to build. The optional dependencies are only needed if you intend to
replicate our experiments. The versions of the dependencies used in testing are
in parenthesis. If for some reason the build fails, please try to install
the versions described prior to opening an issue.

### Required

* GMP (6.2.1-1, 6.0.0-15)
* MPFR (4.1.0.p13-1, 3.1.1-4)
* OpenSSL (1.1.1.k-1, 1.0.2k-21)
* CMake3 (3.21.1-1, 3.17.5-1)
* CUnit (2.1.3)
* Boost (1.76.0-1, 1.53.0-28)
* git (2.33.0-1, 1.8.3-23)

We present the install commands for some Linux distributions:

* Debian/Ubuntu
```bash
sudo apt-get install git cmake libgmp-dev libmpfr-dev libcunit1-dev libboost-all-dev openssl libssl-dev valgrind massif-visualizer
```

* Arch Linux
```bash
sudo pacman -Sy git cmake gmp mpfr cunit boost boost-libs openssl valgrind massif-visualizer
```

* CentOS 8
```bash
sudo dnf install git cmake gmp gmp-devel mpfr mpfr-devel CUnit CUnit-devel boost boost-devel openssl openssl-devel valgrind
```

If your distribution is not listed here make sure you are using the same
dependencies through [pkgs](https://pkgs.org/).

### Optional

`rustup` is a toolchain installer for the Rust programming language. Instead of
using `rustup` to install the required tooling for Rust you may choose to
install `rustc` and `cargo` through the package manager in your Linux
distribution. `hyperfine` is a benchmarking tool written in Rust.
You only need to install `rustup`, or `rustc` and `cargo` if your distro
doesn't provide a binary for `hyperfine`. `numactl` is used to pin a process
to a single core. The repository documentation is provided in the `doxygen`
format. If you want to generate an HTML of the documentation you will have
to install `doxygen`.

* [rustup](https://rustup.rs/) (1.24.3)
* [hyperfine](https://github.com/sharkdp/hyperfine) (1.11.0)
* [numactl](https://github.com/numactl/numactl) (2.0.14-1, 2.0.12-5)
* [doxygen](https://github.com/doxygen/doxygen) (1.8.17, 1.9.1-1)

We present the install commands for some Linux distributions:

* Debian/Ubuntu
    * Using `rustup`
```bash
sudo apt-get install numactl doxygen
cargo install hyperfine
```

    * Not using `rustup`
```bash
sudo apt-get install cargo rustc numactl
cargo install hyperfine
```

* Arch Linux
```bash
sudo pacman -Sy numactl hyperfine doxygen
```

* CentOS 8
    * Using `rustup`
```bash
sudo dnf install numactl doxygen
cargo install hyperfine
```

    * Not using `rustup`
```bash
sudo dnf install numactl cargo rustc
cargo install hyperfine
```

`rustup` will optionally modify your path variable in order to run any
installed binaries. If you changed the `rustup` defaults or did not use
`rustup` to install cargo and rustc, make sure that `$HOME/.cargo/bin` is in
your path. To do so, you can run in bash:
```bash
export PATH=$HOME/.cargo/bin:$PATH
```

## Getting Started

:warning: If you downloaded this zip file from the artifact CHES website,
you don't need to initialize the repository. All thirdparty software,
barring the dependencies described in the previous section, is already
available in the folder `thirdparty`.

After installing all required dependencies, initialize the repo by running:

```bash
./init-repo.sh
```

(Make sure you execute `init-repo.sh` at the root of the artifact.)

:warning: This repository relies on submodules to pull its dependencies.
Therefore, you will need an active internet connection to run `init-repo.sh`.

Our implementations depend on [BLAKE3](https://github.com/BLAKE3-team/BLAKE3).
The `init-repo.sh` will initialize this dependency and connect it to our build
system.

Using cmake, one can build the code using:

```bash
  cmake -DCMAKE_BUILD_TYPE=Release -S. -B_builds -G "Unix Makefiles"
  cmake --build _builds/
```

There are other options the user can use to further specify how the OT and ROT
implementations should be compiled.

* `-DVECTOR_ENGINE={x86 => [AVX512 | AVX2 | SSE | SERIAL]; ARM => [NEON | SERIAL]}`
Specify vector engined to be used. If no value is passed, the `cmake` script
will try and figure out what is the widest vector ISA support in your machine
and default to it.
* `-DNTT_USE_NOISE_CACHE=[ON | OFF]` Use a noise cache when generating random
numbers. Default is off.
* `-DOT_TEST=[ON | OFF]` Compile the OT versions of all algorithms.
Default is off.
* `-DOT_ROTTED_TEST=[ON | OFF]` Compile the ROTed versions of all algorithms.
Default is off. If neither `OT_TEST` or `OT_ROTTED_TEST` are set to ON, `cmake`
will build the standard `ROT` versions. In the case both `OT_TEST` or
`OT_ROTTED_TEST` are set to ON, `cmake` will build the ROTed versions.

Variations to build the code on other systems should be available by consulting
the manpages of `cmake` and changing the `-G` flag accordingly.

Running the compiled (R)OT:
```bash
./_builds/main
```
To run the PVW08 implementation:
```bash
./_builds/pvw/pvw
```

## Docker

If you don't feel like going through all of the above procedures and just want
to see the code compile and run, you can use [Docker](https://www.docker.com/).
To install docker follow the instructions in the
[official documentation](https://docs.docker.com/get-started/).

To create, run, and benchmark all code execute:

```bash
docker build -t rlweot .
docker run -it --rm --name run-rlweot rlweot
```

*Note: Trying to replicate our experiments through docker is not recommended.
Please refer to the section
[Replicating the Experiments](#replicating-the-experiments) in order to better
understand how one should go about it.*

## Reading the Output

By default all programs run through CUnit to ensure that the protocols have
the expected output. As such, the only output available
will be CUnit's output, _i.e._, it will show if any assertions failed.

## Folder Structure

* `include`, `pvw` and `src` contain our implementations.
* `thirdparty` contains third-party libraries which are needed to build the
project.
* `utils` contains utilities scripts to ease certain tasks

## Code Overview

**If you want to reuse our code you should read this section.**
There are 4 relevant folders: `thirdparty/nfl`, `include`, `pvw`, and `src`.

[NFLlib](https://github.com/quarkslab/NFLlib) is the backend used to perform
lattice arithmetic. We have modified `nfl` in order to support an AVX512
backend for x86 architectures and an NEON
backend for ARM architectures. Specifically we added the files
[avx512.hpp](thirdparty/nfl/include/nfl/opt/arch/avx512.hpp) and
[neon.hpp](thirdparty/nfl/include/nfl/opt/arch/neon.hpp) to `thirdparty/nfl/include/nfl/opt/arch/`.
We added the backend support to the operations in
[ops.hpp](thirdparty/nfl/include/nfl/ops.hpp) and architectural support in
[arch.hpp](thirdparty/nfl/include/nfl/arch.hpp). The cmake script was also modified in
order to support the new backends.

`include` contains all of the data structures used in order to run the proposed
ROTs and OTs. The OT implementation is in [rlweot.hpp](include/rlweot.hpp), and
the ROT implementation is in [rlwerot.hpp](include/rlwerot.hpp). The random
oracle implementations are in [roms.hpp](include/roms.hpp). All
implementations are templated in order to facilitate parameters
modifications without sacrificing performance.

`pvw` contains the implementation for the PVW08 proposal. The
implementation uses OpenSSL as a backend for elliptic curve arithmetic.

`src` contains the instantiation and tests for the proposed OT and ROT
implementations.

## Replicating the Experiments

Each algorithm was executed on multiple machines:

* Raspberry Pi 2 (ARM Cortex-A7 @ 900MHz)
* Raspberry Pi 3 (ARM Cortex-A53 @ 1.4GHz)
* Raspberry Pi 4 (ARM Cortex-A72 @ 1.5GHz)
* Apple Mac Mini Late 2020 (Apple M1 @ 3.2GHz)
* A machine with an Intel i9-10980XE @ 3GHz

The first step to replicate the results is to set the clock frequency of all
processors to the described values. There are two scripts in `utils/` which
allow the user to read and set the current operating clock frequency.
The user should run `./utils/watch_cpufreq.sh` to get the available clock
frequencies and `./utils/set_cpufreq.sh` to set one. By setting the
clock frequency to a set value, by default, the "boost" frequency is
disabled.

We strongly encourage people to use the `utils/benchmark.sh` script in order to
replicate the results. This script will build all configurations for the
running architecture and execute all configurations using the benchmark
tool, hyperfine. When executing hyperfine we pin the process to the first core.
This is done in order to ensure that, when a context switch occurs, the process
will be scheduled to run in the same core. Doing so avoids extra cache misses
incurred by scheduling the process to run in a different core. At the end of
the script, hyperfine will output the speedups for each protocol configuration.
Each protocol is executed 1k times. Given that the clock frequency is fixed,
the number of (R)OTs/s = (1k \* (1/FREQ)) / t, where t is the time taken to run
the specified protocol. After benchmarking, the user can reset their
CPU frequency configurations using `./utils/restore_cpufreq.sh`.

The user should not try to replicate results using docker or other
virtualization software. Doing so incurs extra performance penalties due to the
virtualization layer.

_Do not run `./utils/{restore_cpufreq, watch_cpufreq, set_cpufreq}.sh` as root.
All scripts provide a `-h || --help` flag._

### Fixing the Frequency on Intel Machines

If you are getting an error when setting a frequency and you are using
an Intel machine, please go through these steps prior to opening an issue.
A fair degree of caution should be taken when following these steps, as the
changes detailed will fundamentally alter how the CPU operates by setting
a constant frequency. This may be undesireable if, for example,
you are benchmarking on a laptop. Due to the multitude of configurations
present on modern machines, we can't provide a specific step-by-step guide
to everyone. This section only shows the basic steps of fixing the problem.

By default, on Linux, Intel loads a proprietary driver, called `intel_pstate`,
in order to control the clock frequencies of each core. This driver goes against
what the scripts provided try to do. Instead the user should use the Linux
driver `acpi-cpufreq`. To install the driver:

* Debian/Ubuntu
```bash
sudo apt-get install acpi acpid acpi-support
```

* Arch Linux
```bash
sudo pacman -Sy acpi acpid
```

* CentOS 8
```bash
sudo dnf install acpid acpica-tools
```

Next we need to tell Linux not to load that driver. If you are using grub,
edit `/etc/default/grub` and add `intel_pstate=disable` to
`GRUB_CMDLINE_LINUX`. Prior to usage, the grub file needs to be regenerated.
Unfortunately, this step is distro dependent. If you're on Debian/Ubuntu the
command `update-grub` should be available. Otherwise we suggest googling
the correct command for your particular distro.

You may now reboot the machine and access the BIOS. In the BIOS enable the
SpeedStep option. The name of the option will vary by motherboard manufacturer
and, as such, we can't provide more specific details. Once again, if this
specific option isn't available, you're going to have to google for an answer.

After setting the option and reboot again, we can now check whether the
correct driver is being used. For this we will use the utility `cpupower`.

* Debian/Ubuntu
```bash
sudo apt-get install linux-tools-$(uname -r)
```

* Arch Linux
```bash
sudo pacman -Sy cpupower
```

* CentOS 8
```bash
sudo dnf install kernel-tools
```

Running `sudo cpupower frequency-info` should yield

```
analyzing CPU 0:
  driver: acpi-cpufreq
  <snip>
```

The scripts provided should now work.

### Details on `benchmark.sh` and Paper Results

Running `benchmark.sh` creates a folder, `bin`, with all benchmark binaries for
each configuration and a given architecture. For example, in an x86 system (w/
SSE4 and AVX2 support), the following binaries were created in the `bin`
folder, binaries are described in the format
`Binary name: Name in Paper => Description`:

* `pvw_ot`: _SotA [PVW08]_ => The PVW08 implementation;
* `pvw_ot_rotted`: _SotA [PVW08] ROTted_ => The PVW08 implementation, ROTed;
* `avx2_ot`: _RLWE OT (AVX2)_ => Optimized BDGM19 implementation for AVX2;
* `avx2_ot_rotted`: _[BDGM19] ROTted (AVX2)_ => Optimized BDGM19 ROTed implementation for AVX2;
* `avx2_rot`: _RLWE ROT (AVX2)_ => Proposed ROT implementation for AVX2;
* `serial_ot`: _RLWE OT (Serial)_ => Optimized BDGM19 implementation with no vector instructions;
* `serial_ot_rotted`: _[BDGM19] ROTted (Serial)_ => Optimized BDGM19 ROTed implementation with no vector instructions
* `serial_rot`: _[BDGM19] RLWE ROT (Serial)_ => Proposed ROT implementation with no vector instructions
* `sse_ot`: _RLWE OT (SSE4)_ => Optimized BDGM19 implementation for SSE4;
* `sse_ot_rotted`: _[BDGM19] ROTted (SSE4)_ => Optimized BDGM19 ROTed implementation for SSE4;
* `sse_rot`: _RLWE ROT (SSE4)_ => Proposed ROT implementation for SSE4;

On ARM, the following files are created:

* `pvw_ot`: _SotA [PVW08]_ => The PVW08 implementation;
* `pvw_ot_rotted`: _SotA [PVW08] ROTted_ => The PVW08 implementation, ROTed;
* `neon_ot`: _RLWE OT (NEON)_ => Optimized BDGM19 implementation for NEON;
* `neon_ot_rotted`: _[BDGM19] ROTted (NEON)_ => Optimized BDGM19 ROTed implementation for NEON;
* `neon_rot`: _RLWE ROT (NEON)_ => Proposed ROT implementation for NEON;
* `serial_ot`: _RLWE OT (Serial)_ => Optimized BDGM19 implementation with no vector instructions;
* `serial_ot_rotted`: _[BDGM19] ROTted (Serial)_ => Optimized BDGM19 ROTed implementation with no vector instructions
* `serial_rot`: _[BDGM19] RLWE ROT (Serial)_ => Proposed ROT implementation with no vector instructions

An example output of `benchmark.sh` is

```
Benchmark #1: ./avx2_rot
  Time (mean ± σ):      34.4 ms ±   1.1 ms    [User: 33.5 ms, System: 0.9 ms]
  Range (min … max):    33.9 ms …  40.8 ms    1000 runs

Benchmark #2: ./serial_rot
  Time (mean ± σ):      41.0 ms ±   0.7 ms    [User: 40.3 ms, System: 0.8 ms]
  Range (min … max):    40.7 ms …  53.8 ms    1000 runs

Benchmark #3: ./sse_rot
  Time (mean ± σ):      36.1 ms ±   0.1 ms    [User: 35.3 ms, System: 0.8 ms]
  Range (min … max):    36.0 ms …  36.8 ms    1000 runs

Benchmark #4: ./avx2_ot
  Time (mean ± σ):      35.6 ms ±   0.4 ms    [User: 34.8 ms, System: 0.9 ms]
  Range (min … max):    35.3 ms …  40.3 ms    1000 runs

Benchmark #5: ./serial_ot
  Time (mean ± σ):      42.7 ms ±   1.2 ms    [User: 42.1 ms, System: 0.7 ms]
  Range (min … max):    42.4 ms …  56.7 ms    1000 runs

Benchmark #6: ./sse_ot
  Time (mean ± σ):      37.7 ms ±   0.5 ms    [User: 36.9 ms, System: 0.9 ms]
  Range (min … max):    37.5 ms …  48.7 ms    1000 runs

Benchmark #7: ./pvw_ot
  Time (mean ± σ):     491.3 ms ±   8.0 ms    [User: 489.4 ms, System: 1.4 ms]
  Range (min … max):   483.2 ms … 574.4 ms    1000 runs

Benchmark #8: ./pvw_ot_rotted
  Time (mean ± σ):     491.3 ms ±  10.4 ms    [User: 489.3 ms, System: 1.5 ms]
  Range (min … max):   485.0 ms … 651.3 ms    1000 runs

Benchmark #9: ./avx2_ot_rotted
  Time (mean ± σ):      36.2 ms ±   0.5 ms    [User: 35.4 ms, System: 0.9 ms]
  Range (min … max):    35.9 ms …  44.4 ms    1000 runs

Benchmark #10: ./serial_ot_rotted
  Time (mean ± σ):      43.4 ms ±   0.6 ms    [User: 42.7 ms, System: 0.7 ms]
  Range (min … max):    43.1 ms …  55.3 ms    1000 runs

Benchmark #11: ./sse_ot_rotted
  Time (mean ± σ):      38.4 ms ±   0.2 ms    [User: 37.6 ms, System: 0.9 ms]
  Range (min … max):    38.1 ms …  41.9 ms    1000 runs

Summary
  './avx2_rot' ran
    1.04 ± 0.03 times faster than './avx2_ot'
    1.05 ± 0.03 times faster than './sse_rot'
    1.05 ± 0.04 times faster than './avx2_ot_rotted'
    1.10 ± 0.04 times faster than './sse_ot'
    1.12 ± 0.04 times faster than './sse_ot_rotted'
    1.19 ± 0.04 times faster than './serial_rot'
    1.24 ± 0.05 times faster than './serial_ot'
    1.26 ± 0.04 times faster than './serial_ot_rotted'
   14.29 ± 0.54 times faster than './pvw_ot_rotted'
   14.29 ± 0.51 times faster than './pvw_ot'
```

With these results and the (R)OTs/s equation from the previous subsection
((R)OTs/s = (1k \* (1/FREQ)) / t) the user can fill out Tables 2 and 3 of the
paper. Once the results in Table 2 and 3 are available, Figures 3, 4, 5, 6 and
7 can be generated by choosing the baseline and computing the speedups.

Memory results are obtained by running each binary using massif and then
analyzing the peak usage. The commands are
```
valgrind --tool=massif ./BIN_NAME_HERE
massif-visualizer massif.out.XXXXXX
```

### Note on macOS

macOS does not support `numactl` and does not allow to set the clock frequency.
Since we are unable to pin a process to a single core and the clock frequency
varies, the results for this device are pessimistic. You should expect
large variations when benchmarking using this environment.

The benchmarking environment in the Mac Mini should improve once support
for this architecture is merged into the [Linux](https://asahilinux.org/)
kernel.

# OpenSSL Compilation Error

If you are using an old version of OpenSSL, you will get a compilation error
when compiling the PVW08 (`pvw`) implementation.

You can apply the following [patch](utils/openssl_old.patch) in order to fix
it.
```
git apply utils/openssl_old.patch
```

# Auxiliary Documentation

The RLWE OT implementation, with reference [BDGM19] in the paper, is based
on this [paper](https://eprint.iacr.org/2018/1155). The elliptic curve
OT implementation, with reference [PVW08] in the paper, is based on this
[paper](https://eprint.iacr.org/2007/348).

Furthermore, the code has been documented with Doxygen-based comments.

Running

```
doxygen Doxyfile
```

will produce an HTML representation of the comments in the `comments` folder.
To read the documentation, open `index.html` in `comments/html` with your
favorite browser.

## License

The code in this repository, with the exception of the `thirdparty` folder,
can be used under the MIT license. For more information, please refer to
the [license](LICENSE) file.

The code in the `thirdparty` folder can be used under the licenses:
* NFL -> [GPL3](thirdparty/nfl/LICENSE.txt)
* scripts-common -> [GPL3](thirdparty/scripts-common/LICENSE)
* BLAKE3 -> [Apache2](thirdparty/BLAKE3/LICENSE)
