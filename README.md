PyCPP
=====

Python-like C++ environment with an opinionated, yet flexible toolkit for cross-platform C++ development.

> **WARNING** PyCPP is still alpha software. APIs are destined to break, and bugs will certainly exist.

**Table of Contents**

- [Introduction](#introduction)
- [Getting Started](#getting-started)
- [Design](#design)
- [Performance](#performance)
- [Platforms](#platforms)
- [Contributors](#contributors)
- [License](#license)

## Introduction

Modern C++ development depends on a mix of archaic C-libraries with poorly documented interfaces, and template-driven, header-only libraries that produce significant binary and compile time bloat. PyCPP aims to be a collection loosely interconnected features missing from the C++ standard library, boasting lean headers, fast compile times, and permissive licenses, to facilitate app development with minimal overhead.

PyCPP is, in many ways, a spiritual port of Python back to C++. There are idiomatic ways to do certain tasks, and there is a right and a wrong way. "Zero-cost abstractions" don't exist: infinite abstraction leads to unmaintainable complexity. 

PyCPP is a a platform abstraction library built upon a backport of C++17 to C++11-compatible compilers, providing a single, idiomatic, opinionated way to write code. It is designed with ease-of-use and performance in mind.

## Getting Started

// TODO: lols document

## Design

PyCPP is broken down into two core parts: a low-level API that builds and extends upon the C++ standard library, and a high-level API, which recreates Python-like APIs for C++. Each module in the high-level API compile independently of other modules.

The following modules form the low-level API:

1. [Preprocessor](#TODO-link)
2. [System](#TODO-link)
3. [STL](#TODO-link)
4. [SFINAE](#TODO-link)
5. [Adaptor](#TODO-link)
6. [Intrusive](#TODO-link)
7. [Allocator](#TODO-link)
8. [Context](#TODO-link)
9. [Utility](#TODO-link)
10. [Lexical](#TODO-link)

For the low-level utilities, the order is strictly hierarchical in descending order: for example, the `Preprocessor` module should not include any code from the `Lexical` module.

Meanwhile, the remaining modules mimic the Python standard library, forming the high-level API:

// TODO: document


// TODO: need a lot

## Performance

To avoid long build times, PyCPP avoids inlining functions at all costs, and minimizes template definitions within headers, coming with a slight performance penalty. To optimize performance, you should use custom allocators provided in the [pycpp/allocators](/pycpp/allocators), link-time optimization, and the low-level APIs. This should provide comparable or superior performance to most existing libraries, and mitigate the need for explicit inlining.

## Platforms

PyCPP is continually built with the following compiler and compiler versions:

- Clang 3.8+
- GCC 5.3+
- MinGW 5.3.0 (MXE, MinGW, and MSYS2)
- Visual Studio 14 2015
- Visual Studio 15 2017

## Contributors

- Alex Huszagh

## License

Permissive, mostly MIT, see [license](LICENSE.md).
