---
title: ULP and Accuracy
weight: 4

### FIXED, DO NOT MODIFY
layout: learningpathall
---

# ULP Error and Accuracy

We use the ULP Error a metric which computes the difference between two values, a reference and an approximation, relative to how many floating-point “steps” (ULPs) these two numbers are apart.
We use this metric in libamath to compute the accuracy of our functions.

It can be calculated by:

 <!-- to measure the accuracy of our functions in libamath. This is a relative measure of ULP: given a result and a reference, it will tell you how many floating-point “steps” (ULPs) these two numbers are apart. It can be calculated by: -->

```
ulp_err = | x_approx - x_ref | / ULP(x_ref)
```

where:
* `x_approx` is the approximated value
* `x_ref` is the reference value

Because this is a relative measure in terms of floating-point spacing (ULPs) - ie this metric is scale-aware - it is ideal for comparing accuracy across magnitudes. Otherwise, error measure would be very biased by the uneven distribution of the floats.

 <!-- by this expression: 
![Expression to calculate ULP error#center](ulp-error-simple.png "Figure 1. Simple expression to calculate ULP error")

Note that because  -->

<!-- We use this metric to calculate So when we say a certain function f is 3.5 ULP accurate, it means that the f may present up to 3.5 ULP deviation from the result. -->

# ULP Error Implementation

In practise, however, the above expression may take different forms, to account for sources of error that may happen during the computation of the error itself. 
In our implementation, this quantity is held by a term called `bias_correction`:

```
ulp_err = | (x_approx - x_ref) / ULP(x_ref) + bias_correction |
```

This term takes into account two sources of error.

First, error introduced by casting `x_ref` from a higher precision to working precision. This contribution is given in terms of ULP distance:

```
casting_err = | (long_x_ref - x_ref) / ULP(x_ref) |
```

On top of this, in our calculation we are not just interested on how far our approximation is from the correct value.
We are also interested if it's a good approximation to begin with.

Attending to the following diagram

```
Real Line:
   |           |           |           
<--|-----|-----|-----|-----|-----|-->
   |    f0.5   |    f1.5   |    f2.5   
   f0          f1          f2          

Where:
- f1 is a float
- The bin [f0.5, f1.5) is the set of real numbers that round to f1
- The bin width is ULP(f1)
```
In the case where our approximation `x_approx` lands in `f1`, this means `x_approx` could potentially be representing any number in the range: `[f0.5, f1.5)`

Now let's consider the case where the reference `x_ref` is exactly representable by `f0`.
Even though `f0` and `f1` are separated by 1 ULP, we must consider that `f1` could be representing `f0.5`. 
Hence the ULP error in this scenario should be 0.5.

Now let's consider the case where the reference `x_ref` is exactly representable by `f2`.
Following the same logic, `f1` and `f2` are also separated by 1 ULP, but `f1` could be representing `f1.5`. 
Hence in this scenario, the ULP error also amounts to 0.5.

This idea is also factored in our `bias_correction`, by adding or subtracting 0.5, according to the closest possible real number representable by `x_approx`.


```
bias_correction = -casting_err - 0.5 if x_approx > x_ref, -casting_err + 0.5 else

```

Putting all of these ideas together, here is a simplified version of our ULP Error:


```C
// Defines ulpscale(x)
#include "ulp.h"

// Compute ULP error given:
// - got: computed result -> x_approx (float)
// - want_l: high-precision reference -> x_ref (double)
double ulp_error(float got, double want_l) {

    float want = (float) want_l
    // Early exit for exact match
    if (want_l == (double)want && got == want) {
       return 0.0;
    }

    int ulp_exp = ulpscale(want);

    // Fractional tail from float rounding
    double tail = scalbn(want_l - (double)want, -ulp_exp);

    // Difference between computed and rounded reference
    double diff = (double)got - (double)want;

    // Bias for nearest-ULP rounding
    double bias_correction = diff > 0 ? -tail - 0.5 : -tail + 0.5;

    // Return total ULP error with bias correction
    return fabs(scalbn(diff, -ulp_exp) + bias_correction);
}
```

Note that it is possible to get exactly 0.0 ULP error in this implementation if and only if:

* The high-precision reference (want_l, a double) is exactly representable as a float, and
* The computed result (got) is bitwise equal to that float representation.

Here is a small snippet to check out this implementation in action.


```C
#include "ulp.h"

int main() {
    float got = 1.00000001f;
    double want_l = 1.0;
    double ulp = ulp_error(got, want_l);
    printf("ULP error: %.10f\n", ulp);
    return 0;
}
```
The output should be:
```
ULP error: 0.5
```
