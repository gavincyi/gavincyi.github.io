---
layout: post
title: Nogil benchmark
subtitle: Does Sam Gross's nogil implementation perform faster?
tags: [python, nogil, engineering]
comments: true
---

The experiment is to address the following questions

1. Does Sam's nogil version perform as good as it describes?
2. Can it run production application at the moment?
3. Can it be deployed to run production application, before merging back to Python main branch?


## 1. Does Sam's nogil version perform as good as it describes?

### Yes or no.

I used the same performance test from Sam Gross's design [documentation](https://docs.google.com/document/d/18CXhDb1ygxg-YXNBJNzfzZsDFosB5e6BfnXLlejd9l0/edit)
to run both Python 3.9 and nogil's version in [pyperformance](https://pyperformance.readthedocs.io/). Since I got a hard time building the nogil version from the source in my Macbook, I used Github's Action with Docker image to produce the result.

The package pyperformance is originally used for the Python development community to benchmark on read-world examples for all Python implementations. The benchmark includes a few groups. For example, the group `app` tests on high-level applicative components, e.g. Tornado HTTP. 

Regarding to the benchmark result in the document,

> The new interpreter (together with the GIL changes) is about 10% faster than CPython 3.9 on the single-threaded pyperformance benchmarks. 

And when I look back the total elapsed time of my result, the total elapsed time between them are only in modest difference (24 mins with GIL v.s. 25 mins without GIL)

```
python-gil.json
===============

Performance version: 1.0.5
Report on Linux-5.15.0-1019-azure-x86_64-with-glibc2.31
Number of logical CPUs: 2
Start date: 2022-09-23 19:58:04.774574
End date: 2022-09-23 20:21:51.237614

python-nogil.json
=================

Performance version: 1.0.5
Report on Linux-5.15.0-1019-azure-x86_64-with-glibc2.31
Number of logical CPUs: 2
Start date: 2022-09-23 20:24:36.297722
End date: 2022-09-23 20:49:46.225375
```

I am scratching my head.

> A few caveats: First, the performance improvement is an average (geometric mean across ~60 benchmarks). Some benchmarks run much faster and a few run slower than upstream.

Okay that's the trick of benchmarking - geometric mean on the performance improvement

> Second, the new interpreter stripped of the GIL changes would be even faster on single-threaded workloads

My understanding is those changes are merged into Python 3.10 and 3.11.

> Finally, there are still some use cases that don’t scale well, but I think these cases are easier to address with the new design.

Fair enough. We can't expect too much from a preliminary change.

In general, from the result breakdown,

- nogil version performs better in I/O related benchmark, e.g. `tornado_http`, `logging_*`
- nogil performs worse in mathematical computation, e.g. `scimark_sparse_mat_mult`, `telco`

The results addresses getting rid of GIL indeed improves I/O performance, but on the mathematical computation, I doubt biased reference counting and global free-lists overrides the benefit of running without GIL.


In summary, from the benchmark on the proof-of-concept implementation, it opens up both the potential and questions to the project. Will running production applications, like web server / client, give an excellent result without gil?

## 2. Can it run production application at the moment?

### Probably no.

TL;DR Not all the libraries are installable in nogil Python, especially those written with C / C++ extension.

I experienced the following cases when installing packages for benchmark

- When the nogil build wheel can't be found, it needs to download the generic wheel (py2.py3-none-any) and build / compile from scratch. In general it takes a much longer time to install the packages and their dependence.

- Sometimes the compilation involves other compilers, e.g. Rust compiler, in the instance. 

- Only a limited versions are available in C / C++ extension built library. For example, Numpy community has built a few versions of nogil packages (1.18.5, 1.19.3, 1.19.4, 1.22.3)

```
gil
Runtime summary    
=======================
Number: 200
Median: 119
90-pct: 184
95-pct: 226
99-pct: 246
```

```
no-gil
Runtime summary    
=======================
Number: 212
Median: 891
90-pct: 1340
95-pct: 1385
99-pct: 1567
```

```
gil

wrk --duration 30s --threads 10 --connections 300 http://127.0.0.1:8000/ping
Running 30s test @ http://127.0.0.1:8000/ping
  10 threads and 300 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   153.36ms  175.83ms   1.97s    95.84%
    Req/Sec   104.41     44.40   240.00     64.29%
  16462 requests in 30.09s, 2.76MB read
  Socket errors: connect 57, read 141, write 0, timeout 168
Requests/sec:    547.07
Transfer/sec:     94.03KB
```

```
nogil

wrk --duration 20s --threads 1 --connections 100 http://127.0.0.1:8000/ping
Running 20s test @ http://127.0.0.1:8000/ping
  1 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   231.83ms   21.47ms 377.83ms   87.78%
    Req/Sec   430.54     46.91   494.00     87.00%
  8584 requests in 20.04s, 1.44MB read
Requests/sec:    428.33
Transfer/sec:     73.62KB
```


## Reference

https://lukasz.langa.pl/5d044f91-49c1-4170-aed1-62b6763e6ad0/
