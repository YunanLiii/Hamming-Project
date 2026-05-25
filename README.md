# Hamming-Project
Determine the error position in a hamming code
This C++ project demonstrates Hamming code encoding, random bit-flip noise, single-bit error correction, decoding, and a statistics simulation.

## Features

- Encodes binary input using Hamming code with even parity
- Adds random noise with a user-chosen bit-flip probability
- Uses the syndrome to locate a suspected error position
- Corrects one-bit errors
- Extracts the original data bits
- Runs many trials to estimate success and failure rates

## Important idea

A basic Hamming code can correct one flipped bit. If two or more bits flip, the syndrome can point to the wrong position, so the final decoded message may be wrong.

## Compile

```bash
g++ -std=c++17 -Wall -Wextra -pedantic main.cpp -o hamming
```

## Run

```bash
./hamming
```

Then choose:

- `1` for a single demo
- `2` for a statistics simulation

## Example

Input:

```text
1011
0.05
10000
```

The program will simulate random errors and print the success rate.
