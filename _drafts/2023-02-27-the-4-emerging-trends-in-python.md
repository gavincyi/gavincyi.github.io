---
layout: post
title: 2023 Python Trends
subtitle: The 4 emerging trends in Python
cover-img: https://i.pinimg.com/564x/ea/f1/5c/eaf15ca2b1c8a5fa76b95b5ff372a1a1.jpg
tags: [python, cpython]
comments: true
---

## Fading out setup.py (PEP-621)

PEP-518 / PEP-621 is not a new enhancement - it was proposed in 2020 to replace the existing project metadata configuration from `setup.py` to a new TOML format, called `pyproject.toml`. Then it went through a transitional period for pip and setuptools to support TOML configuration. Now, especially in pip v23, setup.py format is regarded as legacy.

```
The interface documented here is retained currently solely for legacy purposes, until the migration to pyproject.toml-based builds can be completed.
```

The main benefit of the migration is, in pyproject.toml format, the build system requirements and information can be specified, and then used by pip to build the package. We will go over the new build concept in the next section.

In 2023, it is definitely a perfect time to start migrating to `pyproject.toml`, if you haven't.

## Independent build process (PEP-517)

The incentive of introducing PEP-517 is to provide flexibility in building packages. Previously, building package is constrainted in `distutils` and `setuptools`. 

1. Declaring the build time dependency
2. Extending the build features
3. Outsourcing the development flow

Basically, the outcome of the proposal is the outflux of packaging and development tools (or ["unintended competition"](https://pradyunsg.me/blog/2023/01/21/thoughts-on-python-packaging/#unintended-competition)) you may have heard recently, like poetry and PDM.

Again, the prerequsite of adopting PEP-517 is `pyproject.toml` migration. For example, you can only use poetry with `pyproject.toml`.

One major difference is 


Also, in latest pip, installing the package with `setup.py` with distutils

## A big step on no-GIL (PEP-703)

## Local packages directory (PEP-582)

