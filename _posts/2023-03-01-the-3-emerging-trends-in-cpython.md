---
layout: post
title: The 3 emerging trends in CPython
subtitle: Which proposals should Python developers keep an eye on in 2023?
cover-img: https://user-images.githubusercontent.com/10500805/222268872-7d436228-253c-42b1-8856-80216f10f1b3.png


tags: [python, cpython]
comments: true
---

Since its first appearance 30 years ago, Python has evolved into a widely adapted language. One of the greatest contributors to its success is CPython governance - its committee reacts swiftly to community requests. Great feature enhancements can be rapidly released in the next version of CPython. (Of course CPython has a speedy schedule to release new versions and deprecate legacy versions at the same time). In 2023, I feel there are 3 emerging trends in CPython worth mentioning, and I believe they are critical for the Python ecosystem competing (and interacting) with other fast-growing programming languages. 

## Fading out setup.py (PEP-621)

[PEP-518](https://peps.python.org/pep-0518/) / [PEP-621](https://peps.python.org/pep-0621/) is not something new - it was proposed in 2020 to replace the existing project metadata configuration from `setup.py` to a new TOML format, called `pyproject.toml`. Then it went through a transitional period for pip and setuptools to support TOML configuration. Now, especially in pip v23, setup.py format is regarded as legacy.

```
The interface documented here is retained currently solely for legacy purposes, until the migration to `pyproject.toml`-based builds can be completed.
```

The main benefit of the migration is, with pyproject.toml format, the build system requirements and information can be specified, and then used by pip to build the package. We will go over the new build concept in the next section.

In 2023, it is definitely a perfect time to start migrating from `setup.py` to `pyproject.toml`, if you haven't.

## Independent build process (PEP-517)

The incentive of introducing PEP-517 is to provide flexibility in building packages. Previously, the building package was constrained in `distutils` and `setuptools`. 

1. Declaring the build time dependency
2. Extending the build features
3. Outsourcing the development flow

Basically, the outcome of the proposal is the outflux of packaging and development tools (or ["unintended competition"](https://pradyunsg.me/blog/2023/01/21/thoughts-on-python-packaging/#unintended-competition)) you may have heard recently, like [poetry](https://pypi.org/project/poetry/) and [PDM](https://github.com/pdm-project/pdm).

Again, the prerequisite of adopting PEP-517 is `pyproject.toml` migration. For example, only packages with `pyproject.toml` can be used in poetry. 

Why is PEP-517 a critical change for Python developers?

Firstly, pip does not handle the build process anymore, but relies on the specified "build backends". The build backends can be setuptools (default), [poetry](https://pypi.org/project/poetry/), [hatchling](https://pypi.org/project/hatchling/), or any backends supporting building C / C++ extensions.

The configuration `pyproject.toml` of the package can (or actually should) specify its build backends. For example, in [fastapi](https://github.com/tiangolo/fastapi/blob/master/pyproject.toml), the configuration specifies `hatchling` as the build backend

```
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

And then pip will proceed the following [procedures](https://pip.pypa.io/en/stable/reference/build-system/pyproject-toml/) to build the package

1. Build isolation: Install the build dependencies into a temporary directory and inject its path into the build commands
2. Generate the package's metadata, if necessary and possible
3. Create a wheel for the package	

Now the build dependency is no longer contained in the system Python / environment, but clearly isolated and discarded after the build.

Secondly, for the packages containing extensions, the extension compilation process can be trivially specified. It is now common to have C / C++ extension for computational intensive packages, e.g. [pandas](https://github.com/pandas-dev/pandas) / [pyarrow](https://github.com/apache/arrow/tree/main/python), or a more state-of-the-art fashion of employing Rust extension, e.g. [polar](https://github.com/pola-rs/polars) / [Pydantic](https://github.com/pydantic/pydantic-core). 

For example, [maturin](https://github.com/PyO3/maturin) is a framework to write Rust extensions. For a Python package with Rust extensions, the project directory looks like the below

```
my-project
├── Cargo.toml
├── python
│   └── my_project
│       ├── __init__.py
│       └── bar.py
├── pyproject.toml
├── README.md
└── src
    └── lib.rs
```

1. `Cargo.toml`, similar to `pyproject.toml`, contains the metadata to compile and build a Rust package.
2. `pyproject.toml` contains the Python metadata, and declares to build the Rust extension specified in `Cargo.toml`

```
[build-system]
requires = ["maturin>=0.14,<0.15"]
build-backend = "maturin"
```

With specifying to use maturin as a build tool, 

> maturin merges metadata from Cargo.toml and pyproject.toml, pyproject.toml take precedence over Cargo.toml.

There are lots of other examples to illustrate how the third-party build tools can now clearly define the building behaviour with `pyproject.toml`. It doesn’t mean this could not be achieved with `setup.py` before, but normally with much painful endeavour by injecting build pipelines. 

In short, there is no direct impact on users to install packages, but it gives great flexibility for developers to choose the right build system for packaging.

## A big step on no-GIL (PEP-703)

[PEP-703](https://peps.python.org/pep-0703/) is an official proposal from Sam Gross's approach to work on a GIL-free Python runtime. Previously, I [wrote](https://gavincyi.github.io/2022-10-03-does-sam-gross-nogil-cpython-fork-perform-faster/) about his amazing approach and its update before the PEP was announced. The ongoing progress with the PEP includes

- It is expected to support no-GIL in CPython 3.13 (around 2024)
- A build time flag `--disable-gil` will be provided to compile the CPython build without GIL. By default, GIL will be retained unless specified. Also, extension packages are required to specify two ABI versions, one with GIL and another without it.
- After 2-3 releases, CPython will be released with GIL controlled by a runtime environment variable or flag. Only a single ABI version is needed.

Every Python developer should keep an eye on it.

## Conclusion

Python started as a weird kid in history while now its development and packaging practices converge with other languages, like Javascript / npm. The first two emerging trends aim at extending Python development experience with other programming languages. A big win for polyglots.

Also, thanks to its flat learning curve, its user base is diversified from web development to computationally driven application, especially machine learning usage. Its evolution balances between I/O and CPU intensive applications. Definitely, ML plays a critical role in the language nowadays. It is interesting to see the no-GIL proposal is mainly [driven](https://peps.python.org/pep-0703/#motivation) by reinforcement learning usage. So the trend is still targeting Python performance, especially more comparable with compiled languages. 

In analyst's terminology, 

> Python still performs strongly due to a diversified market base with continuous adoption of extensions, e.g. C / C++ and Rust. Strong Buy. 


