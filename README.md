# [Team A53 IRIS Labs Hardware] RO Based True Random Number Generator With BIST


Overview: This project implements a secure True Random Number Generator (TRNG) on GF180MCU using thermal noise induced jitter. It features a dual-oscillator entropy source, a [Von Neumann hardware whitener](https://en.wikipedia.org/wiki/Randomness_extractor#Von_Neumann_extractor) to eliminate silicon bias, and a [NIST SP 800-90B compliant BIST](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-90B.pdf) engine for real-time security monitoring. The design integrates via a 32-bit Wishbone interface, providing validated entropy with automatic hardware interrupts upon statistical failure.

1. Architecture
 - Entropy Source (Analog): Dual Ring Oscillators (RO).
 - Digitizer (Mixed-Signal): Dual-stage D-Flip-Flop (DFF) synchronizer acting as a threshold sampler.
 - Post-Processor (Digital): Von Neumann Whitening FSM (https://en.wikipedia.org/wiki/Randomness_extractor#Von_Neumann_extractor)
 - Monitor (Digital): BIST.
 - Wrapper (Digital): 32-bit Wishbone interface with dedicated hardware interrupt.
2. Datapath 
- Generation: Thermal noise generates phase jitter in the Fast RO. Minimum-sized transistors deliberately maximize this noise.
- Sampling: The Slow RO clocks the first Dff, deliberately violating setup/hold times against the Fast RO. The resulting metastable states resolve randomly into 1s and 0s.
- Whitening: The digital FSM groups raw bits into pairs to scrub hardware bias (e.g., 01 becomes 0, 10 becomes 1, while 00 and 11 are discarded).
- Verification: The BIST engine concurrently monitors the whitened stream. It tracks consecutive identical bits (repetition count) and the 1s/0s ratio over a 64-bit window (adaptive proportion).
- Serving: If BIST passes, bits are packed into 32-bit words for the Wishbone bus. If BIST fails, the output register locks and interrupt is raised.
3. Target Specifications 
- Operating Voltage: 3.3V
- Fast RO Frequency: 200 MHz - 400 MHz (for maximum phase jitter)
- Slow RO Frequency: 10 MHz (Current-starved: https://ieeexplore.ieee.org/document/10027058)
- BIST Latency: 1 clock cycle to flag interrupt.
