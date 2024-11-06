# NIST Statistical Test Suite (STS) Version 3

**Note:** Recent changes to this README.md file on **2023 Mar 29**:

- The [Google Drive sts-data folder][generatordata] link has been changed to allow public access.
- Added a "_p.s._" about LandRnd at the bottom.
- Added link to the [improved SP800-22Rev1a paper][improved-paper].

This project is an enhanced version of the [NIST Statistical Test Suite][site] (**STS**), a collection of tests designed to evaluate the randomness of binary sequences. It is widely used for testing random number generators (RNGs) in cryptographic and simulation applications.

## Table of Contents

- [Purpose](#purpose)
- [Applications](#applications)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
  - [Basic Usage](#basic-usage)
  - [Checking Block Devices for Randomness](#checking-block-devices-for-randomness)
  - [Expected Results](#expected-results)
    - [Interpreting P-values](#interpreting-p-values)
    - [NIST Compliance Criteria](#nist-compliance-criteria)
- [Examples](#examples)
  - [Testing a Pseudo-Random Number Generator](#testing-a-pseudo-random-number-generator)
  - [Testing a Hardware Random Number Generator](#testing-a-hardware-random-number-generator)
  - [Testing a Block Device](#testing-a-block-device)
- [Advanced Usage](#advanced-usage)
  - [Distributed Mode](#distributed-mode)
- [Project Structure](#project-structure)
- [Improvements Over Previous Versions](#improvements-over-previous-versions)
- [Future Features](#future-features)
- [Legacy Generators Usage](#legacy-generators-usage)
- [Contributors](#contributors)
- [License](#license)
- [Special Thanks](#special-thanks)

## Purpose

The NIST Statistical Test Suite is designed to:

- Evaluate the randomness of binary sequences produced by hardware or software random number generators (RNGs).
- Provide a comprehensive set of tests for assessing the statistical properties of RNGs used in cryptographic applications.
- Assist in the evaluation of RNGs used in simulation and modeling applications.

## Applications

STS can be utilized in various scenarios, including:

- **Cryptographic Key Generation**: Ensuring that keys are generated with sufficient randomness to prevent predictability.
- **Simulation and Modeling**: Validating the quality of RNGs used in simulations to produce accurate and unbiased results.
- **Hardware RNG Testing**: Evaluating the randomness of hardware-based RNGs, such as those derived from physical phenomena.
- **Entropy Source Assessment**: Assessing the quality of entropy sources used in RNGs.

## Requirements

- **C Compiler**: A standard C compiler (e.g., GCC).
- **FFTW3 Library**: Version 3.3.3 or later of [FFTW3][fftw], required for the Fast Fourier Transform computations.
  - Install via package managers (e.g., `sudo apt-get install libfftw3-dev` on Debian-based systems).
- **Make**: For building the suite.

**Note**: If you cannot install FFTW3, you can compile STS without it using the legacy mode, which uses a slower algorithm.

## Installation

Clone the repository and build the suite:

```sh
$ git clone https://github.com/arcetri/sts.git
$ cd sts/src
$ make
```

If FFTW3 is not available, compile in legacy mode:

```sh
$ make legacy
```

This will generate an executable named `sts_legacy_fft`.

## Usage

### Basic Usage

The STS operates by processing a binary sequence in chunks (bitstreams) of a specified length. The key parameters are:

- **Number of Iterations (`-i`)**: The number of bitstreams to process.
- **Length of a Single Bitstream (`-n`)**: The length of each bitstream in bits (default is 1,048,576 bits).

Ensure that your input data size is at least:

```
(number of iterations) × (length of a single bitstream) / 8 bytes
```

**Example Command:**

```sh
$ ./sts -v 1 -i 32 -I 1 -w . -F r /path/to/random/data
```

**Explanation of Flags:**

- `-v 1`: Sets verbosity level to 1 (optional).
- `-i 32`: Specifies 32 iterations (bitstreams).
- `-I 1`: Reports progress after every iteration (optional).
- `-w .`: Sets the working directory for output files.
- `-F r`: Reads input data as raw binary data.
- `/path/to/random/data`: Path to the input data file.

**Note**: By default, STS uses all available CPU cores. Use `-T 1` to disable multithreading or `-T N` to use `N` threads.

After execution, results are saved in `result.txt`.

**Help Command:**

```sh
$ ./sts -h
```

### Checking Block Devices for Randomness

Block devices (e.g., `/dev/random`, `/dev/urandom`, or hardware RNG devices) can be tested for randomness using STS.

**Example: Testing `/dev/random`**

1. **Collect Data**:

   ```sh
   $ dd if=/dev/random of=random_data.bin bs=131072 count=100
   ```

   This command reads 100 blocks of 131,072 bytes (1,048,576 bits) from `/dev/random`.

2. **Run STS**:

   ```sh
   $ ./sts -v 1 -i 100 -I 10 -w ./results -F r random_data.bin
   ```

   - `-i 100`: Specifies 100 iterations (matching the `count` in `dd`).
   - `-I 10`: Reports progress every 10 iterations.
   - `-w ./results`: Outputs results to the `results` directory.

**Testing a Hardware RNG Block Device**

Replace `/dev/random` with your hardware RNG device path (e.g., `/dev/hwrng`).

### Expected Results

#### Interpreting P-values

Each test produces a P-value indicating the probability that the observed sequence is random:

- **P-value ≥ 0.01**: The sequence passes the test (random).
- **P-value < 0.01**: The sequence fails the test (non-random).

#### NIST Compliance Criteria

For a sequence to be considered random according to NIST standards:

1. **Proportion of Passing Sequences**:

   - At least 98% of the bitstreams should pass each test.
   - For 100 bitstreams, at least 96 should pass.

2. **Uniform Distribution of P-values**:

   - P-values should be uniformly distributed between 0 and 1.
   - Apply a chi-squared test to the P-values to assess uniformity.
   - The P-value of the chi-squared test should be ≥ 0.0001.

**Interpreting Results**:

- **Proportional Failures**: If the proportion of passing sequences is below the threshold, the RNG may be flawed.
- **Uniformity Failures**: If the P-values are not uniformly distributed, there may be underlying patterns.

## Examples

### Testing a Pseudo-Random Number Generator

Use the built-in generators or your own PRNG:

1. **Generate Data**:

   ```sh
   $ ./generators -i 32 1 32 > prng_data.bin
   ```

   - Generates data using Generator 1 (Linear Congruential).

2. **Run STS**:

   ```sh
   $ ./sts -v 1 -i 32 -I 1 -w ./prng_results -F r prng_data.bin
   ```

### Testing a Hardware Random Number Generator

Assuming your hardware RNG is accessible at `/dev/hwrng`:

1. **Collect Data**:

   ```sh
   $ dd if=/dev/hwrng of=hwrng_data.bin bs=131072 count=100
   ```

2. **Run STS**:

   ```sh
   $ ./sts -v 1 -i 100 -I 10 -w ./hwrng_results -F r hwrng_data.bin
   ```

### Testing a Block Device

To test a block device like a hard drive or SSD for randomness:

1. **Collect Data**:

   ```sh
   $ sudo dd if=/dev/sdX of=block_device_data.bin bs=131072 count=100 skip=1000
   ```

   - Replace `/dev/sdX` with your block device.
   - `skip=1000` skips the first 1000 blocks to avoid filesystem metadata.

2. **Run STS**:

   ```sh
   $ ./sts -v 1 -i 100 -I 10 -w ./block_device_results -F r block_device_data.bin
   ```

**Note**: Testing block devices typically reveals non-randomness, as data stored is not random.

## Advanced Usage

### Distributed Mode

For large datasets, STS supports distributed testing across multiple machines.

#### Steps:

1. **Determine Parameters**:

   - Total number of iterations.
   - Number of hosts (machines).

2. **Assign Host Numbers**:

   - Each host gets a unique number from `0` to `N-1`.

3. **Run STS on Each Host**:

   ```sh
   $ ./sts -m i -w /work/dir -j host_number -i iterations -I progress_interval -v 1 /path/to/data
   ```

   - `-m i`: Run in testing mode, outputting P-values.
   - `-j`: Specifies the job number (host number).

4. **Collect P-values**:

   - Gather `.pvalues` files from all hosts into a single directory.

5. **Assess Results**:

   ```sh
   $ ./sts -m a -d /pvalues/dir -w /work/dir -v 1
   ```

   - `-m a`: Run in assessment mode, analyzing collected P-values.
   - `-d`: Directory containing P-values.

**Example**: See the original README for a detailed example with 32 hosts.

## Project Structure

- **src**: Source code of the suite.
- **docs**: Documentation and theoretical papers.
- **tools**: Auxiliary tools and legacy generator code.

## Improvements Over Previous Versions

- **Batch Mode**: Allows automated testing without user intervention.
- **Multithreading**: Utilizes multiple CPU cores for faster processing.
- **Distributed Testing**: Supports testing across multiple machines.
- **Performance Enhancements**: Execution time reduced by approximately 50%.
- **Bug Fixes**: Resolved inconsistencies with the NIST documentation.
- **Code Refactoring**: Improved code structure, readability, and maintainability.
- **Error Handling**: Added comprehensive error checks and messages.
- **64-bit Support**: Enhanced support for modern processors.
- **Documentation**: Extensive code comments and user guidance.
- **Standard Input Support**: Can read test data from standard input.

## Future Features

- **Graphical Visualization**: Integration with Gnuplot for visualizing P-values.
- **New Tests**: Addition of non-approximate entropy tests.

## Legacy Generators Usage

The original 9 generators are available in the `tools` directory as the `generators` tool.

**Generators List**:

1. Linear Congruential
2. Quadratic Congruential I
3. Quadratic Congruential II
4. Cubic Congruential
5. XOR-based Generator
6. Modular Exponentiation
7. Blum-Blum-Shub
8. Micali-Schnorr
9. SHA-1 Based Generator

**Build Generators**:

```sh
$ cd tools
$ make generators
```

**Usage**:

```sh
$ ./generators -i iterations generator_number output_blocks > output_file
```

**Example**:

```sh
$ ./generators -i 64 3 1024 > qc2_data.bin
$ cd ../src
$ make
$ ./sts -v 1 -i 1024 -I 32 qc2_data.bin
```

Refer to the `genall` script in the `tools` directory for more information.

## Contributors

- **Landon Curt Noll** ([lcn2](https://github.com/lcn2))
- **Riccardo Paccagnella** ([ricpacca](https://github.com/ricpacca))
- **Tom Gilgan** ([tgilgs](https://github.com/tgilgs))

We welcome contributions, bug fixes, and improvements. Please submit them via [GitHub Pull Requests](https://github.com/arcetri/sts/pulls).

## License

This software is in the public domain. It was developed at the National Institute of Standards and Technology by employees of the Federal Government in the course of their official duties. Cisco's contributions are also placed in the public domain.

## Special Thanks

We extend our gratitude to the original developers of the NIST STS and the authors of the [original SP800-22Rev1a paper][original-paper]. We recommend using our [improved SP800-22Rev1a paper][improved-paper] for better clarity and understanding.

## LavaRnd p.s.

For an example application of STS, see the [Detailed Description of Test Results and Conclusions from LavaRnd](https://lavarand.com/what/nist-test.html). It provides insights into the ranking methodology, including definitions of proportional failures and uniformity failures.

[site]: http://csrc.nist.gov/groups/ST/toolkit/rng/documentation_software.html
[original-paper]: http://csrc.nist.gov/groups/ST/toolkit/rng/documents/SP800-22rev1a.pdf
[improved-paper]: https://github.com/arcetri/sts/blob/master/docs/SP800-22rev1a-improved.pdf
[fftw]: http://www.fftw.org
[generatordata]: https://drive.google.com/drive/folders/0B-W1rjDbzOiLSVNJWFpkeUE0b1k?resourcekey=0-ogAKUlLH_EvkGEqA461tnQ&usp=sharing
