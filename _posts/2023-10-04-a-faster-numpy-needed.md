---
layout: post
title: A FASTER NumPy Needed
subtitle: How much can we still speed up on NumPy?
thumbnail-img: https://github.com/gavincyi/gavincyi.github.io/assets/10500805/3389a394-c227-4bee-9479-080cc6873788
tags: [numpy, quant, quantdev, finance, gpu, cython]
comments: true
---

Quants always desire for faster computational performance. Is it feasible with a reasonable effort and limited cost?

Prior to the [post](https://gavincyi.github.io/2022-08-27-python-compiler-comparsion-2/) about Python compiler, we have explored Python computation acceleration with compiler libraries, e.g. Numba. The comparison might be a good start for greenfield projects, but it does not sound relevant to those large-scale quantitative libraries. These libraries may have been launched at a time when only NumPy was available as a reliable numerical library, and then scaled up to be used by existing services for a couple of years. Rewriting the whole core computational parts in those libraries is neither economical nor pragmatic. Developers might ask: What are the possible approaches for the NumPy part? What considerations should they have?

## How difficult is it to improve upon the current NumPy performance?

Since the NumPy project (originally called SciPy) started in 2001, the project has undergone a few [stages](https://arxiv.org/abs/1907.10121) of performance improvement.

![U-gELZvwKsBlRTdkt-YLmBuY0psaAolC1VHu0bO9uotw8tWOSDbB6LQTeAEdkBXIIU3BGA634oV0bGL-5FeFY1EZ21s6lsPq5hNbyD-uxQ-pY5d5PErkn0HkAEnl](https://github.com/gavincyi/gavincyi.github.io/assets/10500805/53b3da98-e8d8-47a2-9c4a-9104ffe5bb94)


Since the low-level components of matrix operations in NumPy have been implemented with a Cython interface, C/Fortran-based numerical libraries, like BLAS, LAPACK, and the latest OpenBLAS, handle a vast amount of quantitative computation underneath. There is little room for performance improvement in matrix operations on the CPU level, especially for the large matrices, where the interface overhead is just a marginal difference.

## Vectorization and SIMD

Also, vectorization plays a significant role in NumPy. Along with [SIMD](https://numpy.org/neps/nep-0038-SIMD-optimizations.html#nep38) optimization, NumPy operates element-wise operations in parallel and fits multiple elements into a single CPU instruction. It means that if your function consists of a series of NumPy operations, you are likely taking the advantage of vectorization and running optimised performance in SIMD. There is a slim chance you can rewrite the function within a reasonable time frame to beat NumPy.

We will walk through two main aspects to accelerate NumPy - hardware selection and memory management.

## Hardware selection

NumPy runs on CPU and, at the moment, does not support GPU and other computational hardware, like TPU. This is a major drawback compared to other numerical libraries, e.g. TensorFlow. A wide range of mathematical models in finance and machine learning can be expressed in matrix operations, and GPU expedite them, especially in matrix multiplication. 

Tensorflow and JAX, as mentioned in the previous posts, aim to support multiple runtime hardwares. For example, in Tensorflow, a NumPy experimental module replicates all the existing prototypes in core and linear algebra modules, and converts them into Tensorflow graph operators so that the runtime hardwares can be easily switched. The same goes for CuPy, where the main goal is to support the same compatibility with CUDA low-level libraries on GPUs. The trend is moving towards  users being able to switch between CPU and GPU within the same library / framework.

In my project [factor-pricing-model-risk-model](https://github.com/factorpricingmodel/factor-pricing-model-risk-model), regression is one of the core computational components. Given the equity returns, the factor exposures and returns are converted from each other by regression. The equity universe can scale up to thousands of instruments with several years of daily data. Running the matrix multiplications for regression on a GPU definitely provides significant performance gain.

## Same "NumPy" but on different runtimes

I started an experimental module in the project to allow users to switch from NumPy to another equivalent framework. Meanwhile, retaining NumPy is still vital during development and debugging (as there is still a steep learning curve for TensorFlow, for example). The idea is that users can switch to any specified engines either globally or within a scope. For example, if users want to fit the model in Tensorflow, with a context manager `use_backend`, the engine is switched to Tensorflow at runtime.

```
from fpm_risk_model.engine import use_backend

with use_backend("tensorflow"):
  model.fit(...)
```

I [benchmarked](https://colab.research.google.com/github/factorpricingmodel/factor-pricing-model-risk-model/blob/main/examples/notebook/numpy_backend_engine.ipynb) the performance across various libraries in GPU runtime. Since NumPy does not support GPU runtime, the following chart shows the runtimes between running NumPy in 2 CPU cores v.s. other libraries in 1 NVIDIA T4 GPU. The result aligns with my expectation that GPU runtime brings in significant improvement, especially in larger datasets.

In estimation, if the hourly cost per 1 NVIDIA T4 GPU charges less than 25% of the hourly cost per vCPU, the below performance gain will translate into a lower computational cost. For example, using AWS Sagemaker/Google Colab Notebook can instantly switch from CPU to GPU runtime. After running a smaller dataset for a test run, the instance can then switch to GPU runtime to process larger datasets during training and modelling for optimal FinOps.

![85IxjCZmRcvQIWvHNMehGbfyqXh4bOn20pEM6YvM2XHvcfV-ka5qMKexLLh0VaflTdKO7jYmQAf2JuLaoHMOQ_oeI71aYEgKaQgKvy_NGAIU6o2StqjrLvmfglCR](https://github.com/gavincyi/gavincyi.github.io/assets/10500805/5a35cda3-4c9b-4a10-8fc6-f27883d90aa0)


The bottom line is to provide compatibility in your NumPy operators to run in GPU instances.

## Memory management

One of the keys in NumPy's success is memory storage in NumPy arrays. Python lists are just like C++ vectors of pointers - each item is referred to a memory address but they are not stored in contiguous form. Unlike dynamically typed Python lists, NumPy stores arrays with static types and contiguous form. Storing the array in such a way means faster access in memory (with less [cache misses](https://www.baeldung.com/cs/cache-friendly-code)) and, consequently, a huge difference in runtime.
[Strides](https://numpy.org/doc/stable/reference/generated/numpy.ndarray.strides.html) is a core concept in NumPy array memory to specify the number of bytes in each item. For example, the strides of a float64 array is 8 (8 bytes = 64 bits), while int32 is 4 (4 bytes = 32 bits).

In the meantime, the stride of a 2-d array indicates how many bytes to move forward when iterating through rows and columns. For example, a float64 10x10 array needs to increment 80 bytes and 8 bytes to reach the next row and column respectively.

```
np.zeros((10, 10)).strides

# (80, 8)
```

## NumPy Stride Tricks

NumPy has a specific module `stride_tricks` to create array views with strides without expanding the memory usage. For example, given a 1-d array, the function `sliding_window_view` can expand it into a 2-d array, with each row representing a rolling series.

```
a = np.arange(100)
b = np.lib.stride_tricks.sliding_window_view(a, 10)
b[:2]

array([[ 0,  1,  2,  3,  4,  5,  6,  7,  8,  9],
       [ 1,  2,  3,  4,  5,  6,  7,  8,  9, 10]])
```

Its stride indicates moving only 8 bytes (float64) to iterate through the rows.

```
b.shape

(91, 10)

b.strides

(8, 8)
```

With strides and contiguous alignment, NumPy leverages the BLAS library for running matrix multiplication in multithreading and SIMD instructions. To leverage on parallelization in computation, we can expand a rolling window series into a zero-memory-copy matrix to operate matrix multiplications. For fields, e.g. finance, frequently employing the rolling window algorithm, the trick is definitely a game changer.

A good example is an implementation of a EWM rolling mean on a fixed window. The computation differs from an EWM rolling mean on full series, which has been well implemented in Pandas EWM module. Using pandas `rolling` and `apply` method (iterating each window period to `apply` the vector dot product) is an obvious approach.


```
# Create a time series
a = np.arange(0, 100)
# Convert it into a Pandas Series
s = pd.Series(a)

# Weights of a 10-day EWM rolling window
w = 0.9 ** np.arange(10)
w /= w.sum()
array([0.15353399, 0.13818059, 0.12436253, 0.11192628, 0.10073365,
       0.09066029, 0.08159426, 0.07343483, 0.06609135, 0.05948221])

# Benchmark the Pandas Series performance
%timeit s.rolling(10).apply(lambda x: x @ w)
1.03 ms ± 2.87 µs per loop (mean ± std. dev. of 7 runs, 1,000 loops each)
```

The bottleneck is the single core iterating through the windows, especially when the `apply` function is time-exhausting.

However, when the series can be expanded into a view of matrix, we can leverage matrix multiplication to parallelize the rolling window computation. The benchmark result shows that the improvement in runtime is 1000 times!

```
b = np.lib.stride_tricks.sliding_window_view(a, 10)

%timeit b @ w
1.28 µs ± 1.73 ns per loop (mean ± std. dev. of 7 runs, 1,000,000 loops each)
```

The following graph compares the runtimes on EWM rolling mean on a fixed window (63-days) over various array sizes (from 3-year to 5-year business days). Definitely it is a stunning gain with stride tricks.

![5U5x0IRRCsuDfXmz-u1ZexiFd240krOprLL7SiFVGk-v-VIFtAMdwzlZ9j60ZkVcUnAqnkSWrRAl_ozKd9_5xw7VcRmiUnOiPbjoV1F5mOyA2OuQaSgp6VnwLYq0](https://github.com/gavincyi/gavincyi.github.io/assets/10500805/cbe14a18-5972-4c15-9678-c56860b1c796)


So, the key takeaways are to use contiguous arrays and avoid memory construction. Also, rewriting the low level components, e.g. Pandas rolling series, in numpy stride tricks is a promising strategy to improve performance. 

## Conclusion

In the quest for faster computational performance, we've explored various aspects of accelerating NumPy. While NumPy itself is highly optimized, there are still opportunities for improvement, especially when considering hardware selection and memory management.

In the meanwhile, there are also a few areas to improve the computational runtime. One area is distributed computation, Dask is commonly employed to run computations on  Numpy-like array / Pandas-like dataframe in clusters. Another area is quantization. Machine learning community has successfully developed strategies in training models in lower precision. [Quantized LLM models](https://blog.gopenai.com/the-llm-revolution-boosting-computing-capacity-with-quantization-methods-b8666cdb4b6a), therefore, use fewer resources to perform their operations, making them more efficient and accessible on different hardware platforms. I hope I will walk through these areas in future posts.

![nbUP-i21y0jmvsjycMrh6FEWqNBbWmF4Q801DnqYdXLE4w1SGS7jX9kj3YjEmtrVO2PJx2bm0idpGJBBzV3cImNZ5XlND2dvgXt24TQJGhPD-aXLXZxgbxyhYIre](https://github.com/gavincyi/gavincyi.github.io/assets/10500805/0319c809-9573-44a9-8d85-de5d5e423f22)


In summary, while NumPy remains a powerhouse for numerical computing, exploring hardware options and mastering memory management techniques can further unlock its potential. Consider the specific needs of your projects, and strategically harness the capabilities of modern hardware to supercharge your computational tasks.



