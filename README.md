# ramp-primes: A Large, Random Composite and Prime Number Generator

[![Crates.io](https://img.shields.io/crates/v/ramp-primes)](https://crates.io/crates/ramp-primes)
![Crates.io](https://img.shields.io/crates/l/ramp-primes?style=flat-square)
[![Build Status](https://travis-ci.org/0xAtropine/ramp-primes.svg?branch=master)](https://travis-ci.org/0xAtropine/ramp-primes)


This crate provides a **beautifully simplistic API** for generating large, cryptographically-random, integers in rust, including but not limited to **prime numbers**.

It takes full advantage of the [RAMP](https://crates.io/crates/ramp) crate, which provides high-performance large integers through in-line assembly and a high level, simplistic interface for users to use, as well as lower-level components allowing complete control over large integers.

This crate **requires the use of the nightly toolchain** due to the use of ramp with inline-assembly.

## Usage

Add the following to your `cargo.toml`:

`ramp-primes = "0.4.0"`

`ramp = "0.5"`

If you have not already, install the nightly toolchain as this will only work on the nightly branch and then set as default toolchain.

`rustup toolchain install nightly`

`rustup default nightly`

### Example of Prime Number Generation

```rust
use ramp_primes::Generator;

fn main(){
  // Generates two primes (p,q) both of 512 bits
  let p = Generator::new_prime(512);
  let q = Generator::new_prime(512);
  
  // Generates the modulus n from p and q
  let n = p * q;
}
```

### Example of Safe Prime Number Generation

```rust
use ramp_primes::Generator;

fn main(){
    // Outputs a Large Prime with 64 bits using [(n-1)/2]
    let p = Generator::safe_prime(64);
}
```

### Example of Random Number Generation

```rust
use ramp_primes::Generator;

fn main(){
  // Creates a Large Integer of 1024 bits
  let x = Generator::new_uint(1024);
  
  // Prints out the randomly generated number
  println!("x: {}",x);
}
```

## Prime Number Generation Design

The Prime Number Generation and parts of its code is based on [Snips-AI's Medium Blog on Generating Prime Numbers](https://medium.com/snips-ai/prime-number-generation-2a02f28508ff).

A **conservative attempt** is made at deciding whether a number is prime or not. The number goes through the generation phase and 3 tests to determine if its prime:

### Generation Phase

1. A single parameter is passed to the generator function that indicates the number of bits the prime should be.

2. The userspace CSPRNG is seeded by the operating system to generate the random numbers using the rand crate.

3. An unsigned integer is generated until it passes the prime test, and before being sent to the test, the LSB is changed to 0 to indicate its odd and its MSB is changed to 1 to make sure its the specified bit length.

4. The number is sent to be processed by **three tests**

### Primality Tests

The numbers go through multiple tests to determine whether they are composite or prime.

1. An array of the first 2048 primes is used to check whether the number is divisble by any of the primes in the array.

2. **Fermat's Little Theorem** is performed

3. **Miller-Rabin Primality Test**, the gold standard recommended by the official RSA documentation and by [NIST](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf) on generating primes, is performed with 16 iterations, the same used by Apple's cryptosystem.

If the number passes these tests, it is considered with high probability to be prime. Feel free to verify them yourselves on [Wolfram Alpha](https://www.wolframalpha.com/) by simply typing in the prime number.

### Safe Primes

Safe Primes are generated simply by checking if (p-1)/2 is a prime with the tests listed above.

## License

Licensed under either of

* Apache License, Version 2.0

* MIT license

at your option.

## Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in the work by you, as defined in the Apache-2.0 license, shall be dual licensed as above, without any additional terms or conditions.
