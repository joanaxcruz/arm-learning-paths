---
title: Understanding Multiaccuracy
weight: 5

### FIXED, DO NOT MODIFY
layout: learningpathall
---
# What Is Multiaccuracy?

In amath, multiaccuracy refers to the availability of multiple accuracy levels for the same mathematical function.
This means, some of our functions allow users and compilers to choose between:
* higher accuracy (under 1ULP)
* default accuracy (under 3.5ULP)
* lower accuracy (for max performance where rough results are enough)

# How Is (Multi)accuracy delivered in amath?
You can deduce which accuracy a function is from the last suffix of its name:
<!-- (do I explain the rest of the name?) -->

* `_u10` is the suffix present in high accuracy routine' implementations (for example `_ZGVnN4v_cosf_u10`). These routines present a deviation from the reference no higher than 1 ULP.
* *No* suffix is present in default accuracy provided by libamath (for example `_ZGVnN4v_cosf`). These routines present a deviation from the reference no higher than 3.5 ULP.
* `_umax` is the suffix present in low accuracy routine' implementations (for example `_ZGVnN4v_cosf_umax`). Functions with this suffix yield results where roughly half of the bits are correct. In single precision this corresponds to 4096 ULP error threshold.

# Applications

Choosing the right accuracy level is critical to balancing numerical fidelity and performance. As mentioned before, amath provides three distinct accuracy modes — each optimized for specific application domains. Here’s how to select the most appropriate one for your use case:

## High Accuracy (1ULP)

High accuracy functions in libamath (u10 suffix) present up to 1.0 ULP deviation from the result. 
Hence, this mode prioritizes numerical (almost) correctness.
It ensures minimal error propagation, making it suitable for:
* Scientific computing: Physics simulations, numerical solvers, and finite element methods where small floating-point deviations can cause instability.
* Signal processing pipelines: Where precision impacts filter characteristics, especially in IIR or recursive systems.

With potentially other applications. Performance tends to be lower for these due to more corrective steps (e.g., higher-degree polynomials, range reduction, FMA chains).


## Default Accuracy (3.5ULP)

The default accuracy (no suffix) provided in amath present up to 3.5 ULP deviation from the result. 
It offers a good trade-off between throughput and correctness. It’s typically used for:
* General-purpose libraries: (??Standard math libraries or embedded systems that don’t require strict IEEE 754 compliance but still need reasonable accuracy??);
* Data analytics & feature extraction: Where function calls (e.g., logf, sqrtf) are intermediate steps, and end-to-end accuracy isn't overly sensitive.
* Inference pipelines: Especially on edge devices where latency is important but moderate numerical error is acceptable.

## Low Accuracy / Max Performance (~4096 ULP)

The low accuracy (umax) provided in amath present roughly up to half the bits precision - arund 4096 ULP threshold.
This mode is engineered for aggressive performance, accepting large deviations from the ideal result in exchange for higher throughput. 
* Graphics, games, visual effects: Approximate sin, cos, and exp may be "good enough" for lighting or animation curves.
* Massive input sweeps: Where statistical convergence matters more than individual precision (e.g., Monte Carlo simulations, genetic algorithms).

Not suitable for code paths where small errors can amplify or affect control flow.

# Summary
| Accuracy Mode | amath example          | Approx. Error   | Performance | Typical Applications                                      |
|---------------|------------------------|------------------|-------------|-----------------------------------------------------------|
| `_u10`        | _ZGVnN4v_cosf_u10       | ~1.0 ULP         | Low         | Scientific computing, stable math, backpropagation, validation |
| *(default)*   | _ZGVnN4v_cosf           | ~3.5 ULP         | Medium      | General compute, analytics, inference              |
| `_umax`       | _ZGVnN4v_cosf_umax      | ~4096 ULP      | High        | Real-time systems, embedded DSP, games, visual effects, approximate math |

