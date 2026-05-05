# SEI CERT C++ Vulnerability Remediation

> **Module:** COMP10068 — Secure Programming | UWS  
> **Standard:** SEI CERT C++ Coding Standard  
> **Task:** Analyse, explain, and fix five noncompliant C++17 programs

---

## Overview

Five noncompliant C++ programs were analysed for security vulnerabilities, mapped to the applicable SEI CERT rule, and remediated with compliant fixes. The constraint on every program: the `main()` function could not be modified — fixes had to be implemented within the affected function only.

---

## Program 1 — DCL50-CPP: C-style Variadic Function

**Vulnerability:** The `sum()` function used C-style `va_list` / `va_arg` to accept variable arguments. The compiler cannot verify argument types — passing the wrong type causes undefined behaviour. The function also used `0` as a sentinel to stop reading, meaning `0` could not be passed as a real value, and forgetting the sentinel caused the function to read arbitrary memory.

**Rule violated:** `DCL50-CPP` — Do not define a C-style variadic function.  
**Secondary rule:** `EXP58-CPP` — Pass an object of the correct type to `va_start`.

**Fix:** Replaced with a C++ variadic template using `std::enable_if` and `std::is_integral` to enforce integer-only arguments at compile time. No sentinel value needed — the parameter pack handles termination automatically.

```cpp
// BEFORE — unsafe C-style variadic
int sum(int x, int y, ...) {
    va_list va;
    va_start(va, y);
    while (int v = va_arg(va, int)) { r += v; }
    va_end(va);
}

// AFTER — type-safe variadic template
template<typename T, typename... Rest>
typename std::enable_if<std::is_integral<T>::value, T>::type
sum(T x, Rest... rest) {
    return x + sum(rest...);
}
```

**Result:** `sum(1,2,3,0)` returns `6`. Type errors caught at compile time. No undefined behaviour.

---

## Program 2 — STR50-CPP: Buffer Over-Read

**Vulnerability:** `read()` fills a 32-byte `char` buffer but does not null-terminate it. Passing this buffer to `std::string(buf)` — which expects a null-terminated C-string — causes the constructor to read past the buffer into adjacent memory. This is a buffer over-read: it can leak sensitive data, crash the program, or cause undefined behaviour.

**Rule violated:** `STR50-CPP` — Guarantee that storage for strings has sufficient space for character data and the null terminator.

**Fix:** Replaced the null-dependent constructor with the length-aware `std::string(buf, count)` overload, using `gcount()` to get the exact number of bytes read. Added genuine error detection via `fail()`.

```cpp
// BEFORE — null-dependent, unsafe
std::string str(buf);

// AFTER — length-aware, no null dependency
std::streamsize count = in.gcount();
std::string str(buf, count);  // copies exactly 'count' bytes
```

**Result:** Safe regardless of buffer content. Works correctly even when all 32 bytes are filled with non-null data.

---

## Program 3 — MEM51-CPP: Memory Leak via Manual Deletion

**Vulnerability:** `show()` manually deleted two raw pointer objects. If an exception occurred between the two `delete` statements, the second object would never be freed — a memory leak. In `main()`, both objects were created with `new` inside a function call, meaning if the second allocation failed, the first would never be cleaned up.

**Rule violated:** `MEM51-CPP` — Properly deallocate dynamically allocated resources.

**Fix:** Wrapped both raw pointers in `std::unique_ptr` at the start of `show()`. Memory is released automatically when the pointers go out of scope — whether the function returns normally or via exception.

```cpp
// BEFORE — manual deletion, not exception-safe
void show(Foo *a, Bar *b) {
    std::cout << (a->x + b->y) << std::endl;
    delete a;
    delete b;
}

// AFTER — RAII via unique_ptr, exception-safe
void show(Foo *a, Bar *b) {
    std::unique_ptr<Foo> a_ptr(a);
    std::unique_ptr<Bar> b_ptr(b);
    std::cout << (a_ptr->x + b_ptr->y) << std::endl;
}
```

**Result:** Memory always freed. No performance overhead. Zero changes to `main()`.

---

## Program 4 — MSC51-CPP: Predictable PRNG Output

**Vulnerability:** `std::mt19937` was constructed with no seed argument. Without a seed, it defaults to a fixed value — producing the same sequence of numbers on every run. In security contexts (session tokens, password generation, cryptographic keys), predictable output is exploitable: an attacker who knows the default seed can predict every value the generator will produce.

**Rule violated:** `MSC51-CPP` — Ensure your random number generator is properly seeded.

**Fix:** Seeded `mt19937` with `std::random_device`, which draws entropy from the operating system (hardware RNG on Linux via `/dev/random`, `CryptGenRandom` on Windows).

```cpp
// BEFORE — deterministic, predictable
std::mt19937 engine;

// AFTER — non-deterministic seed
std::random_device rd;
std::mt19937 engine(rd());
```

**Result:** Different sequence on every run. Unpredictable output suitable for security-sensitive contexts.

---

## Program 5 — ERR55-CPP: Violated `noexcept` Specification

**Vulnerability:** Function `f()` was marked `noexcept(true)` — promising it would never throw. But internally it called `v.resize(s)`, which throws `std::bad_alloc` if memory allocation fails. When a `noexcept` function throws, C++ calls `std::terminate()` immediately — the program crashes with no opportunity for error handling. An attacker passing a very large `s` value could force memory exhaustion and crash the program — a denial-of-service vulnerability.

**Rule violated:** `ERR55-CPP` — Honor exception specifications.

**Fix:** Removed `noexcept(true)`. The function now correctly allows exceptions to propagate to callers who can handle them.

```cpp
// BEFORE — lies about exception behaviour
void f(std::vector<int> &v, size_t s) noexcept(true) {
    v.resize(s);  // CAN throw std::bad_alloc
}

// AFTER — honest exception specification
void f(std::vector<int> &v, size_t s) {
    v.resize(s);  // may throw — callers can handle it
}
```

**Result:** `std::bad_alloc` can now propagate correctly. No false `noexcept` contract. Program no longer crashes unconditionally on allocation failure.

---

## Summary

| Program | Rule | Vulnerability Class | Fix Strategy |
|---------|------|--------------------|----|
| noncompliant1.cpp | DCL50-CPP | Type-unsafe variadic, undefined behaviour | Variadic template + `std::enable_if` |
| noncompliant2.cpp | STR50-CPP | Buffer over-read, missing null terminator | `std::string(buf, count)` + `gcount()` |
| noncompliant3.cpp | MEM51-CPP | Memory leak, exception-unsafe deletion | `std::unique_ptr` RAII |
| noncompliant4.cpp | MSC51-CPP | Predictable PRNG, fixed seed | `std::random_device` entropy source |
| noncompliant5.cpp | ERR55-CPP | False `noexcept`, masked `bad_alloc` | Remove false exception specification |

---

## Files

| File | Description |
|------|-------------|
| `B00249469-sp-assignment1-2526_COMPLETED.docx` | Full vulnerability analysis, compliant fixes, and justifications |

---

## Disclaimer

> Assignment completed individually as part of COMP10068 assessed coursework. The `main()` function was not modified in any program, per assignment constraints.
