# Hangman in Rust

> **Module:** COMP10068 — Secure Programming | UWS  
> **Language:** Rust | **Assignment:** Build a fully functional Hangman game from a bare `Hello World` template

---

## Overview

A complete Hangman game built in Rust, demonstrating Rust's memory-safety model, ownership semantics, and standard library patterns. The starting point was a single `main()` function printing `Hello world` — everything else was built from scratch.

---

## How It Works

```
Round starts → load fruits.txt → deduplicate → pick random fruit
     │
     ▼
Display hidden word (underscores for letters, spaces/apostrophes visible)
     │
     ▼
Player guesses letter or full word
     │
     ├─ Correct letter → reveal all matching positions
     ├─ Wrong letter   → deduct attempt, show remaining
     ├─ Full word match → win immediately
     ├─ Full word wrong → deduct attempt, hint: "X letters match"
     └─ Repeated guess  → penalise (no free re-guesses)
     │
     ▼
Win (no underscores remain) or Lose (0 attempts left → reveal word)
     │
     ▼
Play again? (Enter/y → new round, n → exit)
```

---

## Design Decisions

### Duplicate Handling
`fruits.txt` contains repeated entries — Banana appears 10 times, Blood orange appears twice. Rather than editing the file (which was marked uneditable per the brief), entries are collected into a `HashSet` which drops duplicates automatically, then converted to a `Vec` for random selection:

```rust
let unique: HashSet<&str> = contents.lines()
    .map(|l| l.trim())
    .filter(|l| !l.is_empty())
    .collect();
```

### Unicode-Safe Display
The fruit list includes multi-word names (`Blood orange`) and names with apostrophes (`Buddha's hand`). Spaces and apostrophes are revealed from the first turn — hiding them would make the puzzle unsolvable. Letters only are replaced with underscores:

```
Blood orange  →  _____ ______
Buddha's hand →  _______'_ ____
```

### Guess Validation
- All input converted to lowercase immediately — case-insensitive guessing
- Input length determines guess type: single character = letter guess, multiple = word attempt
- Repeated letter guesses are penalised — prevents free enumeration
- Wrong word guesses provide a hint: number of letters that appear in the secret word

### Game Structure
Split into two functions for clean state management:
- `main()` — handles the play-again loop between rounds
- `play_game()` — contains all logic for a single round with its own clean state

---

## Why Rust for Secure Programming

Rust enforces memory safety at compile time — the classes of vulnerabilities fixed in Assignment 1 (buffer over-reads, memory leaks, use-after-free) **cannot exist** in safe Rust:

| C++ Vulnerability (Assignment 1) | Rust Equivalent |
|----------------------------------|-----------------|
| Buffer over-read (STR50-CPP) | Bounds-checked by default — panic on out-of-bounds access |
| Memory leak (MEM51-CPP) | Ownership system — memory freed automatically when owner goes out of scope |
| Use-after-free | Borrow checker — compiler rejects use of moved or dropped values |
| Null pointer dereference | No null pointers — `Option<T>` forces explicit handling of absent values |

This is not just a game — it is a demonstration of what a memory-safe systems language looks like in practice.

---

## Running the Game

```bash
# Ensure fruits.txt is in the project root
cargo run
```

Requirements: Rust toolchain (`rustup`), `rand` crate (declared in `Cargo.toml`)

---

## Files

| File | Description |
|------|-------------|
| `B00249469_ErmandMani_Hangman_solution.pdf` | Assignment report — design decisions, implementation approach, references |

---

## Disclaimer

> Assignment completed individually as part of COMP10068 assessed coursework at UWS.
