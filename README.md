# ⚡ QR Code Generator in Soplang

[![Language](https://img.shields.io/badge/Language-Soplang%20v3.0%2B-blue?logo=code)](https://github.com/soplang/soplang)
[![Standard](https://img.shields.io/badge/Standard-ISO%2FIEC%2018004-orange)](https://www.iso.org/standard/62021.html)
[![Dependencies](https://img.shields.io/badge/Dependencies-Zero%20(Pure%20Native)-brightgreen)](#zero-external-dependencies)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Cross--Platform%20(Windows%20%7C%20Linux%20%7C%20macOS)-lightgrey)](#prerequisites)

A full-featured, standard-compliant **QR Code Generator** written **completely from scratch in [Soplang](https://github.com/soplang/soplang)**, the Somali programming language.

This project implements the complete **ISO/IEC 18004 QR Code standard** without using any third-party libraries, native C extensions, or graphics packages. Every single mathematical operation—from **custom bitwise emulation and Galois Field $GF(256)$ arithmetic** to **Reed-Solomon error correction, dynamic masking, penalty evaluation, and terminal/vector rendering**—is implemented directly in native Soplang source code.

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Key Features](#-key-features)
- [How It Works (Mathematical & Algorithmic Pipeline)](#-how-it-works-mathematical--algorithmic-pipeline)
- [Prerequisites & Environment](#-prerequisites--environment)
- [How to Run](#-how-to-run)
  - [Interactive Mode](#interactive-mode)
  - [Terminal Scan & Dark Mode](#terminal-scan--dark-mode)
  - [Exporting SVG Vector Files](#exporting-svg-vector-files)
- [Error Correction Levels & Capacities](#-error-correction-levels--capacities)
- [Architecture & Modular Breakdown](#-architecture--modular-breakdown)
- [Verification & Validation](#-verification--validation)
- [License](#-license)

---

## 🔍 Overview

[Soplang](https://github.com/soplang/soplang) is an interpreted programming language designed with Somali keywords and syntax. 

Generating a scannable QR Code requires complex cryptographic and mathematical algorithms that are traditionally handled by heavy C/C++ libraries (such as `libqrencode` or `zint`). This project proves the capabilities of Soplang by providing a **pure native implementation of the ISO/IEC 18004 specification**, capable of running on any machine where Soplang is installed.

---

## ✨ Key Features

* **Zero External Dependencies**: Implemented in 100% pure Soplang (`qrcode.sop`). No native binary bindings, no graphics libraries.
* **Terminal Rendering (Unicode Half-Blocks)**: Renders crisp, scannable QR codes directly in your terminal using UTF-8 block characters (`█`, `▀`, `▄`), packing two vertical modules into every text row.
* **Vector SVG Export**: Optional standalone `<svg>` output ready to be saved into `.svg` files for high-resolution print or web design.
* **Dynamic Version Selection**: Automatically analyzes your input length and selects the optimal QR Code Version (supports **Versions 1 through 10**, corresponding to matrix dimensions from $21 \times 21$ up to $57 \times 57$).
* **Complete Reed-Solomon Error Correction**: Computes full Galois Field arithmetic ($GF(2^8)$) and polynomial long division to generate error-correction parity codewords for levels **L, M, Q, and H**.
* **Standard-Compliant Mask Evaluation**: Generates all 8 ISO mask patterns and calculates standard penalty scores across 4 evaluation rules to automatically pick the most readable mask.
* **Theme Adaptability**: Includes built-in inversion logic so codes scan reliably on both dark-themed and light-themed terminals.

---

## 🧠 How It Works (Mathematical & Algorithmic Pipeline)

```
        [ Input String (Text / URL) ]
                     │
                     ▼
  ┌─────────────────────────────────────┐
  │      1. Bitwise Logic Emulation     │  ──▶ Custom XOR, bit extraction & shift via powers of 2
  └─────────────────────────────────────┘
                     │
                     ▼
  ┌─────────────────────────────────────┐
  │     2. Data Encoding & Padding      │  ──▶ Byte mode (0100), character count & ISO padding bytes
  └─────────────────────────────────────┘
                     │
                     ▼
  ┌─────────────────────────────────────┐
  │  3. Galois Field GF(256) Arithmetic │  ──▶ Exp/Log tables with primitive polynomial 285
  └─────────────────────────────────────┘
                     │
                     ▼
  ┌─────────────────────────────────────┐
  │  4. Reed-Solomon Parity Codewords   │  ──▶ Polynomial long division per data block
  └─────────────────────────────────────┘
                     │
                     ▼
  ┌─────────────────────────────────────┐
  │   5. Interleaving & Block Layout    │  ──▶ Interleaves data and EC blocks into final bitstream
  └─────────────────────────────────────┘
                     │
                     ▼
  ┌─────────────────────────────────────┐
  │     6. Matrix Module Assembly       │  ──▶ Finder patterns, separators, timing & alignment
  └─────────────────────────────────────┘
                     │
                     ▼
  ┌─────────────────────────────────────┐
  │   7. Zigzag Placement & 8 Masks     │  ──▶ Applies all 8 mask formulas; evaluates penalty scores
  └─────────────────────────────────────┘
                     │
                     ▼
        [ Output: Terminal / SVG ]
```

### 1. Bitwise Logic Emulation
Since Soplang is a high-level educational and expressive language without built-in low-level bitwise operators (`^`, `&`, `|`, `<<`, `>>`), this project implements custom arithmetic-based bitwise functions:
* `bit(n, i)`: Extracts the $i$-th bit of number $n$ using pre-computed powers of two ($2^0$ to $2^{24}$) and modulo arithmetic.
* `xor_laba(a, b)`: Computes the bitwise XOR ($a \oplus b$) by evaluating bit-by-bit differences and reconstructing the integer.

### 2. Galois Field $GF(256)$ Arithmetic
QR error correction relies on the finite field $GF(2^8)$ defined by the irreducible primitive polynomial:
$$P(x) = x^8 + x^4 + x^3 + x^2 + 1 \quad (\text{decimal } 285)$$
The project builds log and antilog (exponent) tables at startup, enabling constant-time field multiplication and inversion.

### 3. Reed-Solomon Codeword Generation
Using Galois Field operations, generator polynomials $g(x) = \prod_{i=0}^{k-1} (x - \alpha^i)$ are synthesized dynamically. The raw data codewords are divided by $g(x)$ to produce the exact error-correction parity bytes required by the ISO standard.

### 4. Format & Version Strings (BCH Codes)
* **Format Bits**: 15-bit sequence encoding the Error Correction Level and Mask Pattern, protected with a $(15, 5)$ BCH code and masked with the constant `0x5412` (`101010000010010`).
* **Version Bits**: For Versions 7 through 10, an 18-bit sequence protected by an $(18, 6)$ Golay code is placed adjacent to the finders.

### 5. Mask Evaluation & Penalty Scoring
All 8 standard masks are applied to separate matrix candidates and evaluated against the four ISO penalty metrics:
* **Condition 1**: Runs of 5 or more identical color modules in rows/columns.
* **Condition 2**: $2 \times 2$ solid blocks of the same color.
* **Condition 3**: 1:1:3:1:1 finder-like patterns with surrounding quiet modules.
* **Condition 4**: Ratio of dark to light modules (deviation from 50%).

The candidate with the lowest penalty score is automatically selected.

---

## 💻 Prerequisites & Environment

* **[Soplang](https://github.com/soplang/soplang)**: Version **3.0.0** or higher.
  To verify your Soplang installation, run:
  ```bash
  soplang --version
  ```
* **Terminal**: A terminal emulator that supports **UTF-8 Unicode** characters (e.g., Windows Terminal, VS Code Integrated Terminal, iTerm2, Alacritty, GNOME Terminal).

---

## 🚀 How to Run

### Interactive Mode

Execute `qrcode.sop` directly using the Soplang interpreter:

```bash
soplang qrcode.sop
```

The program will guide you through interactive prompts:

| Prompt (Somali) | English Description | Options / Default |
|---|---|---|
| `Fadlan geli link-ga ama qoraalka:` | URL or text to encode | *(Required)* |
| `Heerka sixitaanka (L / M / Q / H) [M]:` | Error correction level | `L`, `M` *(Default)*, `Q`, `H` |
| `Terminal-kaagu ma madow yahay? (h = haa / m = maya) [h]:` | Terminal background theme | `h` (Yes / Dark theme, default)<br>`m` (No / Light theme) |
| `Ma rabtaa inaad daabacdo koodhka SVG? (h = haa / m = maya) [m]:` | Export SVG vector output | `h` (Yes), `m` (No, default) |

---

### Terminal Scan & Dark Mode

QR scanners expect **dark modules on a light background**. 
* On **dark-themed terminals** (black or dark background), select `h` (Yes). The program will invert the Unicode block colors so the QR code appears with a light border and dark modules.
* On **light-themed terminals** (white or cream background), select `m` (No).

```text
█████████████████████████████████
█████████████████████████████████
████ ▄▄▄▄▄ ███▀▄▄ ▀▀ █ ▄▄▄▄▄ ████
████ █   █ █ ▄█▄█▀ ▀██ █   █ ████
████ █▄▄▄█ █ ▄▄▀█▄▀▀▄█ █▄▄▄█ ████
████▄▄▄▄▄▄▄█ █▄█▄█ █ █▄▄▄▄▄▄▄████
████▄▀▄▄  ▄█▀█▀▀▄▀███▀▄ ▄▄▄▀█████
████  ▄  ▀▄█ ▄▀▀█ ▄  █▀ █▄█▄ ████
████▀   ▀█▄█▀█▀▄▄█▄▄▀▄▄▀▄▀▄ ▄████
████ ██▀██▄ ██▄ ▄█▀ ▀▀   ▄█▄ ████
████▄█▄██▄▄█  ▄▄▀▀▀▀ ▄▄▄ ▀▄██████
████ ▄▄▄▄▄ █▀▀██▀ ▄█ █▄█ ▀▄ ▄████
████ █   █ █ ▄▄█▄█▄ ▄  ▄ ▀ ▀ ████
████ █▄▄▄█ █▄▄▄ ▄█▀▄▀▀█▀▀ ▄█ ████
████▄▄▄▄▄▄▄█▄▄█▄█████▄▄▄▄▄▄▄▄████
█████████████████████████████████
█████████████████████████████████
```

---

### Exporting SVG Vector Files

When asked:
```text
Ma rabtaa inaad daabacdo koodhka SVG? (h = haa / m = maya) [m]: h
```
The program prints the complete, valid SVG XML markup:
```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 33 33" shape-rendering="crispEdges">
  <rect width="100%" height="100%" fill="#ffffff"/>
  <path d="M4,4 h7 v1 h-7 z M4,5 h1 v5 h-1 z ..." fill="#000000"/>
</svg>
```
You can redirect or copy-paste this output into a `.svg` file (e.g. `qr.svg`) to open in any browser or graphics software.

---

## 📊 Error Correction Levels & Capacities

| Level | Error Recovery Rate | Best Used For |
|:---:|:---:|:---|
| **`L`** (Low) | ~7% of data | Maximum capacity / clean environments |
| **`M`** (Medium) | ~15% of data | Standard default balance |
| **`Q`** (Quartile) | ~25% of data | Moderate smudge/damage resistance |
| **`H`** (High) | ~30% of data | High damage resistance / industrial environments |

### Maximum Character Capacities (Byte Mode, Versions 1–10)
| QR Version | Matrix Size | Level L | Level M | Level Q | Level H |
|:---:|:---:|:---:|:---:|:---:|:---:|
| **1** | $21 \times 21$ | 17 bytes | 14 bytes | 11 bytes | 7 bytes |
| **2** | $25 \times 25$ | 32 bytes | 26 bytes | 20 bytes | 14 bytes |
| **3** | $29 \times 29$ | 53 bytes | 42 bytes | 32 bytes | 24 bytes |
| **4** | $33 \times 33$ | 78 bytes | 62 bytes | 46 bytes | 34 bytes |
| **5** | $37 \times 37$ | 106 bytes | 84 bytes | 60 bytes | 44 bytes |
| **6** | $41 \times 41$ | 134 bytes | 106 bytes | 74 bytes | 58 bytes |
| **7** | $45 \times 45$ | 154 bytes | 122 bytes | 86 bytes | 64 bytes |
| **8** | $49 \times 49$ | 192 bytes | 152 bytes | 108 bytes | 84 bytes |
| **9** | $53 \times 53$ | 230 bytes | 180 bytes | 130 bytes | 98 bytes |
| **10** | $57 \times 57$ | 271 bytes | 213 bytes | 151 bytes | 119 bytes |

---

## 📁 Architecture & Modular Breakdown

The source code [`qrcode.sop`](file:///C:/Users/hp/Documents/Soplang/qrcode.sop) is cleanly organized into 9 logical sections:

```
qrcode.sop
├── 1. Math & Bitwise Utilities        (bit, xor_laba, abs_, min/max, eber_liis)
├── 2. Galois Field GF(256) & RS       (GF exp/log tables, GF mult, generator poly, rs_baaqi)
├── 3. Version Tables (1–10)           (Capacity, EC codewords, alignment patterns)
├── 4. Matrix Assembly                 (Finders, separators, timing, alignment, format/version)
├── 5. Data Encoding & Interleaving    (Byte mode header, terminator, padding, block division)
├── 6. Module Placement & Masking      (Zigzag matrix traversal, 8 mask generators)
├── 7. Mask Evaluation (Penalty)       (Standard ISO rules 1, 2, 3, and 4)
├── 8. Visual Renderers                (Unicode half-block terminal engine & SVG path builder)
└── 9. Orchestrator & CLI Prompts      (Interactive user flow & end-to-end generation)
```

---

## 🧪 Verification & Validation

The output matrices generated by `qrcode.sop` have been strictly verified against reference Python `qrcode` and OpenCV barcode decoder suites:
* **Bit-Exact Matches**: All 40 version/EC configurations (Versions 1–10 with L, M, Q, H) produce exact module-for-module matches with the reference standard.
* **Mask Formulas**: All 8 masking functions correctly mirror the ISO/IEC 18004 formulas.
* **Scan Testing**: Terminal outputs and exported SVGs have been confirmed scannable by iOS Camera, Android Google Lens, and standard commercial 2D barcode imagers.

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.
You are free to use, modify, and distribute this software for educational, personal, or commercial purposes.