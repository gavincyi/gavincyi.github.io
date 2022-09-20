---
layout: post
title: Python Compiler Comparison (1)
subtitle: Cython, Numba and JAX
tags: [python, cython, numba, jax, engineering]
comments: true
---

It has been a while I haven't looked into the current development of Python compilers of which improves the running time of critical paths, e.g. in quantitative finance / scientific research, in the application. There are much more libraries tackling the problem, either by bridging the original CPython C extension library, or by just-in-time (JIT) implementation. I will first go through with three prevailing libraries nowadays.

## Cython, Numba and JAX

First it comes with Cython, the backbone of Pandas, to give easy access to C extension with Python-alike syntax. Then Numba takes advantage of modern LLVM development to compile every CPython bytecode operator on the fly. Finally JAX from Google joined the game in 2018. It first focused on the gradient / differentiation in deep learning, and now there are more industrial use cases to use its JIT compilation functionality.

## Benchmark function

I picked a function to compute the weighted sum of numerical integral. The integral is sliced in arbitrary and the slices come as an input parameter.

![image](https://user-images.githubusercontent.com/10500805/191352088-81589ddd-2ace-4e41-8df9-95f1276b3c65.png)


I first wrote the function in Python, and supposed it is the most straight-forward approach which developers / researchers employs. The numpy sum method actually runs in C extension underneath. Cheating! But let's see how it goes in the comparison afterward.

![image](https://user-images.githubusercontent.com/10500805/191352121-0ac3fee2-376d-49c9-91d6-c4c3f4b1bd3a.png)

## Cython

For Cython, same as my previous experience, first I had to convert the native Python function to Cython syntax, with defining the function as "cpdef" (Cython and Python accessible). Secondly I defined all the types of the function parameters, variables and outputs. Finally I compiled and prayed for it producing a great result. In general, I reckoned that I had to abandon all the numpy built-in functions (as simple as sum), and rewrote the algorithm (e.g. by for loop).

The Python run spends about 2 ms per loop, while, after a long endeavour, Cython runs it in microsecond level. Not bad.

![image](https://user-images.githubusercontent.com/10500805/191352233-36ebee2d-4d8b-4c1b-b0bb-be554a3338dc.png)


## Numba

The life in Numba is much enjoyable. I can directly compile the native Python function with its helper function. I didn't bother too much about turning the parameters in Numba.

![image](https://user-images.githubusercontent.com/10500805/191352256-5d0f9db6-3734-4561-a028-f147e364242f.png)

And the performance is amazing, as good as the Cython implementation.

![image](https://user-images.githubusercontent.com/10500805/191352270-8e002113-db00-43e5-b84b-98d9db2213ee.png)


## JAX

Finally, there are a few considerations in running JAX benchmark. First it is needed to convert the numpy array into the JAX array, and the convention overhead is excluded from the benchmark result. Second, the JAX operation dispatches the result in asynchronously, so it requires to block and wait for the complete result to arrive. Thirdly, the result in the example is a single float but stored in JAX array. Though the convention to Python native type does not incur much overhead, it is excluded in the benchmark as well.

![image](https://user-images.githubusercontent.com/10500805/191352304-bfc5d904-64a9-4358-91bf-d9b74f329cc7.png)

Finally, there are a few considerations in running JAX benchmark. First it is needed to convert the numpy array into the JAX array, and the convention overhead is excluded from the benchmark result. Second, the JAX operation dispatches the result in asynchronously, so it requires to block and wait for the complete result to arrive. Thirdly, the result in the example is a single float but stored in JAX array. Though the convention to Python native type does not incur much overhead, it is excluded in the benchmark as well.

In my example, the Python implementation slices the array in dynamic, i.e. the slicing index is originated from the user input and not determined in compilation time. JAX does not support compilation in dynamic array slicing at the moment. To work around, a partial function is constructed by passing the slicing array (t_data) into the compilation. The compilation takes a while (around 2 mins when the size of the slicing array is 1000), and I suppose the compiled result is large.

The performance time improves a lot, though it is not as fast as the Cython and Numba ones.

![image](https://user-images.githubusercontent.com/10500805/191352371-28329b90-dc80-48fb-9eb0-c6f3403b8228.png)


## Which one is the best?

Does it conclude that Cython is still the best library to compile Python code to C extension level? I would like to answer the question in a few levels.

### 1. Runtime

The first level is on the machine performance time. Actually it is an illusion that Cython gives the best performance from my example. If I rewrite the native Python function like the Cython implementation (use for loop to iterate rather than numpy.sum), the Numba compiled function produces the best performance.

![image](https://user-images.githubusercontent.com/10500805/191352422-3f98516f-634e-4e8e-9bc5-d08799b8f49b.png)

Also, Numba has spent a vast amount of efforts to support numpy features. I believe in a lot of cases, especially those involving numpy arrays, Numba outperforms Cython in a significant margin.

### 2. Development effort

The second level is the development time. Numba has a great benefit and assurance to users to keep the existing Python code and then attempt to give improvements in runtime. The investment of development time is minimal in Numba, while the reward can be huge. This feature has outperformed Cython and JAX.

### 3. Long term maintenance

The third level is the long term adaption to revolutionary technology. Numpy and Cython can only support CPU runtime, while Numba and JAX have already adapted GPU. JAX even supports TPU, a critical advantage to machine learning practitioners. The asynchronous dispatch feature in JAX echoes with the modern machine learning development to optimise the CPU and GPU usage in clusters.

## Reference
The benchmark notebook can be accessible in [Google Colab](https://colab.research.google.com/drive/1DLb8SgR7S4UCbpW1YDmlS0EWruB-Z2Ki?usp=sharing). Open for comments and ideas!

(Original post was in [Linkedin](linkedin.com/pulse/python-compiler-comparison-1-gavin-chan/))
