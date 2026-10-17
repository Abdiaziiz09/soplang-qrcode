# QR Code Generator in Soplang

A pure QR code generator written from scratch in [Soplang](https://github.com/soplang/soplang), the Somali programming language.

This project implements the complete QR Code specification (ISO/IEC 18004) without external libraries or dependencies—implementing custom bitwise operations, Galois Field arithmetic, Reed-Solomon error correction, masking algorithms, penalty scoring, and rendering directly in Soplang.

---

## Features

- **Zero External Dependencies**: Implemented purely in native Soplang (`qrcode.sop`).
- **Terminal Rendering**: Displays scannable QR codes directly in your terminal using UTF-8 block characters (`█`, `▀`, `▄`).
- **SVG Export**: Optional raw SVG vector output that can be saved and viewed in any browser or vector editor.
- **Dynamic Sizing**: Automatically selects the optimal QR version (Versions 1 to 10) based on input length.
- **Custom Math & Encoding**: Built-in bitwise logic emulation, GF(256) Galois Field tables, and Reed-Solomon error-correction code generator.
- **Standard Compliant**: Evaluates all 8 QR masking patterns and applies the best one using standard penalty score rules.

---

## Prerequisites

- **[Soplang](https://github.com/soplang/soplang)** (v3.0.0 or higher)
- **UTF-8 Compatible Terminal** (to properly render Unicode block characters)

---

## How to Run

Run the script using the Soplang interpreter:

```bash
soplang qrcode.sop
```

### Interactive Prompts

The program interactively prompts for the link or text (press **Enter** to accept the default values):

| Prompt | Description | Options / Default |
|---|---|---|
| `Fadlan geli link-ga ama qoraalka:` | URL or text to encode | *(Required)* |
| `Heerka sixitaanka (L / M / Q / H) [M]:` | Error correction level | `L`, `M` *(default)*, `Q`, `H` |
| `Terminal-kaagu ma madow yahay? (h = haa / m = maya) [h]:` | Terminal background theme | `h` (Yes / Dark theme, default)<br>`m` (No / Light theme) |

---

### Example

```text
Fadlan geli link-ga ama qoraalka: https://example.com
Heerka sixitaanka (L / M / Q / H) [M]: M
Terminal-kaagu ma madow yahay? (h = haa / m = maya) [h]: h

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

## Error Correction Levels

| Level | Error Recovery Capacity | Data Capacity |
|:---:|:---:|:---:|
| **`L`** | ~7% | Highest |
| **`M`** | ~15% | High *(default)* |
| **`Q`** | ~25% | Medium |
| **`H`** | ~30% | Lowest |

---

## Tips & Usage Notes

- **Terminal Scanning**: QR codes require dark modules on a light background. Select `h` on dark terminals (colors will be automatically inverted) and `m` on light terminals. If your scanner has difficulty reading the code, zoom in or increase the terminal font size.
- **Exporting SVG**: When prompted, choose `h` to print the full `<svg ...>...</svg>` markup. You can copy-paste this output into a `.svg` file (e.g., `qr.svg`) to open or edit it in any graphics software or web browser.
- **Input Specifications**:
  - Encodes standard ASCII/Byte mode data.
  - Supports Versions 1 through 10. Maximum character capacities:
    - Level `L`: up to 271 characters
    - Level `M`: up to 213 characters
    - Level `Q`: up to 151 characters
    - Level `H`: up to 119 characters
  - Input text is automatically trimmed of leading and trailing whitespace.

---

## Architecture Overview

All implementation logic resides in [`qrcode.sop`](file:///c:/Users/hp/Documents/Soplang/qrcode.sop), structured into modular sections:

| Section | Description |
|---|---|
| **Configuration** | Mask overrides (`MASK_TIJAABO`: `-1` for auto selection, `0`–`7` for manual test mask). |
| **1. Math Helpers** | Bit manipulation, custom XOR emulation (`xor_laba`), absolute values, min/max utilities. |
| **2. Galois Field & Reed-Solomon** | GF(256) exponent/log tables, GF multiplication, generator polynomial creation, and error-correction codeword generation (`rs_baaqi`). |
| **3. Capacity Tables** | Data block counts, EC codeword counts, and alignment pattern coordinates for Versions 1–10. |
| **4. Matrix Assembly** | Finder patterns, separators, alignment patterns, timing patterns, and format/version information bits. |
| **5. Data Encoding & Interleaving** | Byte mode encoding, length indicators, terminator/padding bytes, block splitting, and codeword interleaving. |
| **6. Data Placement & Masking** | Zigzag data module placement and application of the 8 standard QR mask formulas. |
| **7. Mask Evaluation (Penalty)** | Implementation of the 4 standard QR penalty rules to select the optimal mask. |
| **8. Rendering** | Terminal text renderer (`tus_terminal`) using block characters and vector generator (`samee_svg`). |
| **9. Main Routine** | User prompt input flow and end-to-end orchestration. |

---

## Verification & Testing

The generator's output has been comprehensively validated against the reference Python `qrcode` library and decoded using OpenCV:

- All 40 matrix configurations (Versions 1–10 across error levels L, M, Q, H at maximum capacity) matched the reference module-for-module.
- All 8 masking pattern implementations were tested and verified against reference matrices.
- Exported SVGs decode reliably back to original URLs across multiple standard barcode readers.