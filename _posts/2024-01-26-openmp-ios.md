---
layout: post
title: iOS上的 OpenMP 支持
subtitle: #pragma omp parallel
categories: OpenMP iOS 并行
tags: [OpenMP, iOS, 并行]
---

## iOS 上的 OpenMP 支持

### 遇到的问题

最近在编译 iOS 下的 [rapidsnark](https://github.com/0xPolygonID/rapidsnark)
发现运行起来对比 Android 性能低了不少

检查了 [CMakeLists.txt 中的如下片段](https://github.com/0xPolygonID/rapidsnark/blob/9327f3c6a8626f4dc2803d5caea4743981689776/CMakeLists.txt#L34C1-L51C1)，发现在 iOS 上就没有启用 openmp 支持

```makefile
if(USE_OPENMP)
    find_package(OpenMP)

    if(OpenMP_CXX_FOUND)
        if(TARGET_PLATFORM MATCHES "android")
            message("OpenMP is used")

        elseif(CMAKE_HOST_SYSTEM_NAME STREQUAL "Linux")
            message("OpenMP is used")

        else()
            set(OpenMP_CXX_FOUND FALSE)
            message("OpenMP is not used")

        endif()
    endif()
endif()
```

### 网上的说法

按网上的说法，因为 Apple 推荐用 GCD 的方式处理并行需求，所以对 openmp 在 iOS 上的支持不足。
用`brew install libomp`, 也只能安装 macOS 的 openmp library。

用 xcode 自带的 clang 运行`clang -fopenmp  -E -dM - < /dev/null`
会提示`clang: error: unsupported option '-fopenmp'`

## OpenMP 的工作方式

```cpp
template <typename Field>
void FFT<Field>::reversePermutation(Element *a, u_int64_t n) {
    int domainPow = log2(n);
    #pragma omp parallel for
    for (u_int64_t i=0; i<n; i++) {
        Element tmp;
        u_int64_t r = BR(i, domainPow);
        if (i>r) {
            f.copy(tmp, a[i]);
            f.copy(a[i], a[r]);
            f.copy(a[r], tmp);
        }
    }
}
```

注意其中的`#pragma omp parallel for`, 这和 GCD 或者多线程/进程开发的方式有些不同，看起来不需要开发者显示告诉编译器并行优化逻辑，而是让
