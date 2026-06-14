# RobotBurst

[![RobotBurst](robotburst.jpg)](https://github.com/williamstaffordparsons/robotburst)

## Table of Contents

- [Introduction](README.md?tab=readme-ov-file#introduction)
- [Author](README.md?tab=readme-ov-file#author)
- [License](README.md?tab=readme-ov-file#license)
- [Implementation](README.md?tab=readme-ov-file#implementation)
- [Parallelism](README.md?tab=readme-ov-file#parallelism)
- [Randomness Test Results](README.md?tab=readme-ov-file#randomness-test-results)
- [Cryptanalysis Results](README.md?tab=readme-ov-file#cryptanalysis-results)
- [Speed Test Results](README.md?tab=readme-ov-file#speed-test-results)

## Introduction

RobotBurst is a hyper-efficient cryptographic PRNG (for 64-bit architecture) that has a minimum period of at least 2⁶⁴, arbitrary jumps (by using a counter and a fixed key), excellent randomness test results, hyper-fast speed, low-footprint implementation, parallelism and quantum resistance (PQC).

## Author

RobotBurst was created by [William Stafford Parsons](https://github.com/williamstaffordparsons) as a product of [SeedFlurry](https://seedflurry.github.io).

## License

RobotBurst is licensed with the [BSD-3-Clause](LICENSE) license.

## Implementation

RobotBurst was implemented in C (requiring the `stdint.h` header to define a 64-bit, unsigned integral type for `uint64_t`).

[robotburst.c](robotburst.c)

#### `robotburst256`

The `robotburst256` function modifies the state in a `struct robotburst256_state` instance to generate 4 pseudorandom `uint64_t` integers in the `output` array.

Each single-letter variable (`a`, `b`, `c`, `d`, `e` and `f`) in a `struct robotburst256_state` instance must be [seeded](https://seedflurry.github.io) before generating a deterministic `robotburst` sequence.

#### `robotburst512`

`robotburst512` doesn't exist yet (approximately 85% faster than `robotburst256`).

#### `robotburst1024`

`robotburst1024` doesn't exist yet (approximately 150% faster than `robotburst256`).

## Parallelism

RobotBurst enables a set of up to 2⁶⁴ parallel sequences (that each have non-probabilistic full state collision avoidance with each other for a period of at least 2⁶⁴).

Each instance within a set of parallel RobotBurst instances must adhere to the following single-letter variable seeding rules before generating parallel RobotBurst sequences.

#### Method A

1. `a` must be [seeded](https://seedflurry.github.io) with an integer that's unique among the set of parallel RobotBurst instances.
2. `b` must be [seeded](https://seedflurry.github.io) with an integer that's consistent among the set of parallel RobotBurst instances.
3. The remaining single-letter state variables must be [seeded](https://seedflurry.github.io).

After seeding a set of parallel RobotBurst instances, reseeding (or tampering with) either `a` or `b` voids the aforementioned parallelism segmentation properties.

#### Method B

1. `a` must be [seeded](https://seedflurry.github.io) with `0` (and incremented before generating each RobotBurst `output` result).
2. `b`, `c`, `d`, `e` and `f` must be [seeded](https://seedflurry.github.io) (and assigned as a fixed key before generating each RobotBurst `output` result) with integers that are unique among among the set of parallel RobotBurst instances.

Furthermore, `a` is the counter (starting at `0`) of each generated RobotBurst `output` result within a RobotBurst sequence, therefore enabling arbitrary jumps (backward and forward) to any index (by assigning the index to `a` before generating a RobotBurst `output` result) in the fixed-key RobotBurst sequence.

## Randomness Test Results

RobotBurst yields excellent results from statistical test suites for randomness quality.

RobotBurst uses straightforward chaotic state accumulation procedures to reduce the probability of hidden statistical biases from specific seeded states (that each have a minimum period of at least 2⁶⁴ from a Weyl sequence).

The following RobotBurst randomness test results seeded each state variable with `0` (not reseeding or tampering with RobotBurst state variables after seeding).

#### PractRand 0.96

RobotBurst had no `FAIL` results from `stdin -tlmax 32TB -tlmin 1KB` tests.

#### Dieharder 3.31.1

RobotBurst had no `FAILED` results from `dieharder -Y 1 -a -g 200 -k 2` tests.

## Cryptanalysis Results

#### Bit Independence Criterion (BIC)

RobotBurst had excellent results from a BIC test that uses PractRand 0.96 `stdin -tlmax 32TB -tlmin 1KB` tests to evaluate correlations among a worst-case output sequence (with equidistributed indices of the flipped state bit) of XORed `output` integer pairs (each generated from a state pair that differs by 1 bit).

#### Strict Avalanche Criterion (SAC)

RobotBurst had an excellent SAC score of between 0.499 and 0.501 among several thousand worst-case variations (with equidistributed indices of both the flipped state bit and the `output` bit) of a SAC test that counts the occurence of `1` bits among 2²⁰ XORed `output` bit pairs (each generated from a state pair that differs by 1 bit).

#### Next-Bit Test (Forward Unpredictability)

Instead of using underlying quantum-vulnerable primitives with classical-hard mathematical foundations, RobotBurst uses the straightforward combination of BIC and SAC to strongly indicate asymptotic next-bit security from unpredictable input-to-output mapping.

#### State Compromise (Backward Secrecy)

RobotBurst has 256 bits of security (in the single-letter variables `c`, `d`, `e` and `f`) protecting previous states from attackers by requiring at least 2²⁵⁶ brute-force guesses (computationally infeasable) to either attempt reversing the `mix` accumulation in the `while` loop or determining the state from the mixed `output` bits.

## Speed Test Results

Each of the following results log the fastest process execution speed (in milliseconds) among several repetitions of a CSPRNG speed test (using `gcc -O3` from an AMD A4-9120C) that generates (and hashes) 1 billion pseudorandom `uint64_t` integers in a `#pragma GCC unroll 0` loop.

| PRNG | Elapsed |
| --- | --- |
| **`robotburst1024`** | **?ms** |
| **`robotburst512`** | **?ms** |
| **`robotburst256`** | **3494ms** |
| ***`aes_ni_ctr_128` (`-maes -msse4`) | 3796ms |
| **`google_randen` (`-maes -msse4`) | 9206ms |
| *`chacha8` | 13230ms |
| *`chacha20` | 26402ms |

\* Each `uint32_t` output integer was casted to a `uint64_t` integer.

\** Each block of 4 `uint8_t` integers were casted to a `uint64_t` integer.

\*** Each 128-bit output integer was extracted as 2 `uint64_t` integers.
