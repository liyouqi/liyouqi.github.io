---
title: "Trapezoidal Rule with OpenMP: critical and reduction"
date: 2025-11-07
categories:
  - High-Performance Computing
tags:
  - Reflection
  - Journal
  - HPC
  - OpenMP
layout: single
author_profile: true
read_time: true
comments: false
share: true
---

Recently I was practicing OpenMP with a simple numerical integration problem. The task itself is not difficult, but it helped me understand why `reduction` is useful.

I first wrote it with `critical`, then changed it to `reduction`. Here are my notes about these two versions.

## The trapezoidal rule

I used the function

$$
f(x)=x^2
$$

on the interval $[0,1]$. We already know its exact integral:

$$
\int_0^1 x^2 dx = \frac{1}{3}.
$$

So it is easy to check whether the program gives a reasonable answer.

The trapezoidal rule divides the interval into $n$ small parts. The width of each part is

$$
h=\frac{b-a}{n}.
$$

![Trapezoidal rule](/assets/images/HPC/1.png)

For the implementation, I used this form of the formula:

$$
I \approx h\left(\frac{f(a)+f(b)}{2}+\sum_{i=1}^{n-1}f(a+ih)\right).
$$

The two endpoints are calculated only once. Most of the work is the loop in the middle, and this is the part I want to parallelize.

## Serial version

Before using OpenMP, I wrote a serial version as a reference.

```c
#include <stdio.h>

double f(double x) {
    return x * x;
}

int main() {
    double a = 0.0;
    double b = 1.0;
    int n = 1000000;

    double h = (b - a) / n;
    double sum = (f(a) + f(b)) / 2.0;

    for (int i = 1; i < n; i++) {
        double x = a + i * h;
        sum += f(x);
    }

    double result = h * sum;
    printf("Integral (serial) = %.15f\n", result);

    return 0;
}
```

The output should be close to `0.333333333333`. This version is also useful later when checking the parallel result.

## My first OpenMP version

My first thought was to put `omp parallel for` before the loop. But all threads would update the same `sum` variable:

```c
#pragma omp parallel for
for (int i = 1; i < n; i++) {
    double x = a + i * h;
    sum += f(x);
}
```

This has a race condition. For example, two threads may read the same old value of `sum`, add their own value, and then overwrite each other.

To avoid this, each thread can keep a `local_sum`. When its loop is finished, it adds the local result to the shared `sum` inside a critical section.

```c
#include <stdio.h>
#include <omp.h>

double f(double x) {
    return x * x;
}

int main() {
    double a = 0.0;
    double b = 1.0;
    int n = 1000000;

    double h = (b - a) / n;
    double sum = (f(a) + f(b)) / 2.0;

    #pragma omp parallel
    {
        double local_sum = 0.0;

        #pragma omp for
        for (int i = 1; i < n; i++) {
            double x = a + i * h;
            local_sum += f(x);
        }

        #pragma omp critical
        {
            sum += local_sum;
        }
    }

    double result = h * sum;
    printf("Integral (critical) = %.15f\n", result);

    return 0;
}
```

The important point is that `critical` is outside the loop. If I put it around `sum += f(x)` for every iteration, the threads would spend a lot of time waiting. Here, each thread enters the critical section only once.

This code works, but I need to create the local variable and combine it manually.

## Using reduction

Then I learned that OpenMP already has a construct for this kind of calculation:

```c
#pragma omp parallel for reduction(+:sum)
```

With `reduction(+:sum)`, every thread gets its private copy of `sum`. OpenMP combines these copies with `+` after the loop. It is basically the same idea as `local_sum`, but the code is shorter.

```c
#include <stdio.h>
#include <omp.h>

double f(double x) {
    return x * x;
}

int main() {
    double a = 0.0;
    double b = 1.0;
    int n = 1000000;

    double h = (b - a) / n;
    double sum = (f(a) + f(b)) / 2.0;

    #pragma omp parallel for reduction(+:sum)
    for (int i = 1; i < n; i++) {
        double x = a + i * h;
        sum += f(x);
    }

    double result = h * sum;
    printf("Integral (reduction) = %.15f\n", result);

    return 0;
}
```

I prefer this version because `reduction(+:sum)` directly shows what the loop is doing. There is no need to manage `local_sum` or write a critical section myself.

I compiled the programs with OpenMP enabled:

```bash
gcc -fopenmp trap_critical.c -o trap_critical
gcc -fopenmp trap_reduction.c -o trap_reduction

OMP_NUM_THREADS=1 ./trap_reduction
OMP_NUM_THREADS=4 ./trap_reduction
OMP_NUM_THREADS=8 ./trap_reduction
```

The results from different thread numbers should stay close to $1/3$. The last few digits may be slightly different because floating-point additions can happen in a different order.

## What I got from this exercise

The `critical` version was helpful for understanding what happens between the threads. Each thread calculates a partial sum, and the critical section protects the final update.

For a normal summation loop, however, `reduction` is easier to read and less error-prone. OpenMP can handle the private values and the final combination for me.

I have not made a proper performance test for these versions yet, so I do not want to make a speed comparison only from one run. The next step is to run them on the cluster with different thread numbers and record the execution time. It will also be interesting to see when the parallel overhead becomes larger than the calculation itself.
