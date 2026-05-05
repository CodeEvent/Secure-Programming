# Secure Programming — COMP10068

![C++17](https://img.shields.io/badge/C%2B%2B-17-blue?logo=cplusplus)
![Rust](https://img.shields.io/badge/Rust-1.x-orange?logo=rust)
![SEI CERT](https://img.shields.io/badge/Standard-SEI%20CERT%20C%2B%2B-red)
![UWS](https://img.shields.io/badge/UWS-BEng%20Cyber%20Security-darkblue)

Two assessed assignments demonstrating secure coding in two distinct paradigms — systematic vulnerability remediation against the SEI CERT C++ standard, and memory-safe application development in Rust.

---

## Assignments

### [Assignment 1 — SEI CERT C++ Vulnerability Remediation](./SEI-CERT-CPP-Remediation/)

Five noncompliant C++ programs analysed, explained, and fixed against the SEI CERT C++ Coding Standard. Covers type safety, memory management, string handling, PRNG seeding, and exception specification correctness.

| Rule | Vulnerability Class | Fix |
|------|--------------------|----|
| DCL50-CPP | C-style variadic — type-unsafe, undefined behaviour | C++ variadic template + `std::enable_if` |
| STR50-CPP | Buffer over-read — missing null terminator | Length-aware `std::string(buf, count)` |
| MEM51-CPP | Memory leak — exception-unsafe manual deletion | `std::unique_ptr` RAII |
| MSC51-CPP | Predictable PRNG — unseeded `mt19937` | `std::random_device` entropy source |
| ERR55-CPP | Violated `noexcept` — masked `std::bad_alloc` | Remove false exception specification |

### [Assignment 2 — Hangman in Rust](./Hangman-Rust/)

A fully functional Hangman game built in Rust from a bare `Hello World` template. Demonstrates Rust's memory-safety model, ownership semantics, and standard library patterns — with deliberate design decisions around duplicate handling, Unicode-safe display, and guess validation.

---

## Why This Matters for Security

**C++ SEI CERT** — The vulnerabilities fixed here are classes that cause real-world exploits: type confusion, buffer over-reads, memory corruption, predictable token generation, and crash-on-exception denial of service. Understanding why each is dangerous — not just how to fix it — is what separates a security engineer from a developer.

**Rust** — Rust eliminates entire classes of memory safety vulnerabilities at compile time. No use-after-free, no buffer overflows, no null pointer dereferences — by design. Working in Rust demonstrates awareness of why memory safety matters and how a modern systems language enforces it.

---

> **Module:** COMP10068 — Secure Programming | University of the West of Scotland  
> BEng (Hons) Cyber Security | Student ID: B00249469
