# Steganography & Hidden Data Detection — Digital Forensics Home Lab

A hands-on forensics exercise demonstrating LSB steganography embedding/extraction using `steghide`, integrity verification via SHA-256 hashing, and password-dictionary recovery attacks using `stegseek`.

**Author:** Zeeshan Sajid
**Environment:** Kali Linux (VM)
**Date:** July 2026

---

## Objective

Investigate how steganography conceals data inside an ordinary-looking carrier file, and demonstrate — from a forensic investigator's perspective — why visual inspection alone cannot detect hidden data, and why weak steganography passwords are trivially breakable.

Steganography differs from encryption in a key way: encryption scrambles data so it's unreadable without a key, but its *presence* is usually obvious. Steganography hides the *existence* of the data itself inside an innocuous carrier (e.g. a BMP image), so an investigator who isn't already suspicious may never think to look.

## Tools Used

| Tool | Version | Purpose |
|---|---|---|
| `steghide` | 0.6.0 | Embed/extract hidden data in BMP carrier files |
| `stegseek` | 0.6 | Dictionary-based password recovery against steghide files |
| `file` | 5.47 | File type / header identification |
| `xxd` | — | Hex-level byte inspection |
| `strings` | — | Extract readable text from binary data |
| `sha256sum` (GNU coreutils) | 9.10 | Cryptographic integrity verification |

## Methodology

**1. Environment setup** — Verified connectivity, updated package lists, installed and version-checked all required tools.

**2. Carrier & payload preparation** — Downloaded a BMP carrier image (1365×2265, 24-bit color) and created a short evidence text file. SHA-256 hashes of both were recorded as an integrity baseline before any modification.

**3. Embedding** — Used `steghide embed` to hide the secret file inside the BMP, producing a new stego image while leaving the original carrier untouched:
```bash
steghide embed -ef secret.txt -cf original.bmp -sf stego.bmp -p <password>
```

**4. Extraction & verification** — Extracted the hidden payload with the correct password and confirmed byte-for-byte recovery using `diff` and matching SHA-256 hashes.

**5. Visual/binary comparison** — Compared original vs. stego image at file size, BMP header, and `strings` level. Both were visually and structurally identical — same size, identical header bytes — yet hashed completely differently, proving embedding alters data at the byte level without any visible trace.

**6. Password dictionary attack (defensive demo)** — Built a small custom wordlist and ran `stegseek` against the stego image. The password was recovered in seconds:
```
[i] Found passphrase: "****"
[i] Original filename: "secret.txt"
[i] Extracting to "stego.bmp.out"
```

**7. Independent second run** — Repeated the full embed → extract → hash-verify → dictionary-crack workflow end-to-end with a self-authored note and a self-chosen password, to validate the process independently of the guided example.

## Hash Verification Table

| File | SHA-256 (truncated) |
|---|---|
| Original carrier image | `6508bebb94bd8ec0833b9f85580cda4...` |
| Original secret file | `d51feb39a0ee55fe5a22da5df177aa4...` |
| Stego image (post-embed) | `1b2402e1296ae0690db6794ddea150e...` |
| Extracted secret file | `d51feb39a0ee55fe5a22da5df177aa4...` ✅ matches original |
| Independent task note (original vs. re-extracted) | `13b71d80bf9ea84d265bfe6d8aec577...` ✅ identical |

Matching hashes confirm lossless recovery in both runs. The differing hash between the original and stego image — despite identical size and header — confirms embedding altered the underlying pixel data without any visible or structural change.

## Key Findings

- **Visual/structural inspection is insufficient.** Identical file size, identical BMP header, indistinguishable appearance — yet completely different SHA-256 hashes. Only hash comparison against a known-good original (or an active extraction attempt) reveals hidden data.
- **LSB steganography leaves no readable trace.** Running `strings` against the stego file returned high-entropy binary noise, not contiguous text, since the payload is spread bit-by-bit across pixel data.
- **Weak passwords defeat steganography entirely.** Both a generic weak password and a "personal-sounding" password (name + numbers) were recovered by `stegseek` in seconds against a small, targeted wordlist — the same risk that makes rockyou.txt-style attacks effective at scale in the real world.
- **Hashing, not appearance, is the reliable evidence of file integrity** in a forensic context.

## Relevance to GRC / Security Practice

This exercise reinforces principles directly relevant to information security and compliance work:
- Data integrity verification methodology (supports evidentiary chain-of-custody thinking)
- Practical understanding of a data-exfiltration/concealment technique relevant to DLP and insider-threat controls
- Real demonstration of why password policy strength matters — not just in theory, but as an exploitable control gap

---
*This repository documents a personal, self-directed home-lab exercise conducted in an isolated VM environment for skill-building purposes. No real or sensitive data was used.*
