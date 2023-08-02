---
description: Why do you need it?
---

# The significance

While fuzzing is a powerful testing technique, and a considerable step up from the plain unit testing, it still has some limitations that motivate the need for a complementary symbolic execution of the test suite. Due to pseudo-random input generation, fuzzing struggles to generate input values for complex or nested conditions. Symbolic execution, systematically explores all feasible code paths by using symbolic variables as input, tracking path conditions. This provides a more comprehensive coverage.

In addition to this, symbolic execution can automatically derive and check postconditions, providing stronger guarantees on the correctness of the program. These complementary approaches can be used to identify a wider range of bugs and vulnerabilities, leading to more reliable software.

By installing **Kontrol**, you unlock the capability to perform property verification! This is a big step up in assurance from property testing, but is more computationally expensive, and often requires manual intervention.
