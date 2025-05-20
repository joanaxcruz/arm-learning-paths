---
title: Examples
weight: 6

### FIXED, DO NOT MODIFY
layout: learningpathall
---

# Example

Here is an example invoking all multiaccuracy variants of the expf function:

```C { line_numbers = "true" } 
#include <amath.h>
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

#include "ulp_error.h"

int main(void) {
    // Inputs that trigger worst-case errors for each accuracy mode
    float argmax_umax = -0x1.5b7322p+6;
    float argmax_u10  = 0x1.791298p+2;
    float argmax_u35  = 0x1.8163ccp+5;

    // Broadcast to vector
    float32x4_t vargmax_umax = vdupq_n_f32(argmax_umax);
    float32x4_t vargmax_u10  = vdupq_n_f32(argmax_u10);
    float32x4_t vargmax_u35  = vdupq_n_f32(argmax_u35);

    // Evaluate expf with different accuracy modes
    float32x4_t vef_u10  = _ZGVnN4v_expf_u10(vargmax_u10);
    float32x4_t vef      = _ZGVnN4v_expf(vargmax_u35);      // default
    float32x4_t vef_umax = _ZGVnN4v_expf_umax(vargmax_umax);

    // Extract first element
    float ef_u10  = vgetq_lane_f32 (vef_u10, 0);
    float ef      = vgetq_lane_f32 (vef, 0);
    float ef_umax = vgetq_lane_f32 (vef_umax, 0);

    // Use double-precision exp() as reference
    double want_u10  = exp((double)argmax_u10);
    double want_u35  = exp((double)argmax_u35);
    double want_umax = exp((double)argmax_umax);

    // Output results
    printf("AMath example: Multi Accuracy in action\n");
    printf("-----------------------------------------------\n");
    printf("  // Display bits that are guaranteed to be correct\n");
    printf("  // in each accuracy mode when using vector expf.\n");

    printf("  _ZGVnN4v_expf_u10(%.6a) = %.12a with error < 1.0 ULP\n", argmax_u10, ef_u10);
    printf("                               ref = %.12a\n", want_u10);
    printf("                         ULP error = %.4f\n\n", ulp_error(ef_u10,  want_u10));

    printf("  _ZGVnN4v_expf_u10(%.6a) = %.12a with error < 3.5 ULP\n", argmax_u35, ef);
    printf("                               ref = %.12a\n", want_u35);
    printf("                         ULP error = %.4f\n\n", ulp_error(ef,  want_u35));

    printf("  _ZGVnN4v_expf_u10(%.6a) = %.12a with half correct bits\n", argmax_umax, ef_umax);
    printf("                               ref = %.12a\n", want_umax);
    printf("                         ULP error = %.4f\n\n", ulp_error(ef_umax,  want_umax));

    return 0;
}
```

You can compile the above program with:
```bash
gcc -O2 -ffast-math -o example example.c -lamath -lm
```

Running the example should return:
```bash
$ ./example 
AMath example: Multi Accuracy in action
-----------------------------------------------
  // Display bits that are guaranteed to be correct
  // in each accuracy mode when using vector expf.
  _ZGVnN4v_expf_u10(0x1.791298p+2) = 0x1.6a0ab8000000p+8 with error < 1.0 ULP
                               ref = 0x1.6a0ab8183f88p+8
                         ULP error = 0.4526

  _ZGVnN4v_expf_u10(0x1.8163ccp+5) = 0x1.6a09e0000000p+69 with error < 3.5 ULP
                               ref = 0x1.6a09e3e3d585p+69
                         ULP error = 1.4450

  _ZGVnN4v_expf_u10(-0x1.5b7322p+6) = 0x1.9b56be000000p-126 with half correct bits
                               ref = 0x1.9b491b9376d3p-126
                         ULP error = 1744.7120
```

The inputs we use for each variant correspond to the worst case scenario known to date (ULP Error argmax).
This means that the ULP error should not be higher than the one we demonstrate here, meaning we stand below the thresholds we define for each accuracy.