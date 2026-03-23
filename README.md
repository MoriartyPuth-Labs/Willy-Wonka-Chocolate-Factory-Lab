# ChocolateFactory.exe - Detailed Technical Writeup

<img width="1024" height="559" alt="image" src="https://github.com/user-attachments/assets/67274fa0-66e0-4832-9a82-4b29938681b1" />

A detailed technical analysis and keygen implementation for `ChocolateFactory.exe`, a 64-bit Windows console application challenge from crackmes.one. This writeup covers initial reconnaissance, anti-debugging measures, and the four-stage validation logic used to verify the "Golden Ticket."

## 🖥️ Machine Specifications
* **Target Binary**: `ChocolateFactory.exe` 
* **Platform**: crackmes.one 
* **Format**: PE32+ executable (console) x86-64, Windows
* **Protections**: Anti-debug (IsDebuggerPresent + PEB.NtGlobalFlag), timing gate 
* **Download**: https://crackmes.one/download/crackme/69b4768cf2d49d8512f649ff
* **Password**: crackmes.one 

---

## Technical Analysis

### Phase 1: Initial Reconnaissance
The binary was analyzed to determine its architecture and identify embedded strings.
* **Command**: `file ChocolateFactory.exe` 
* **Result**: 64-bit Windows PE binary with 6 standard sections (.text, .rdata, .data, .pdata, .fptable, .reloc).
* **Strings**: The binary contains UI prompts for a 16-character "Golden Ticket" and identifies four validation "stations": Cocoa Plantation, Milk River, Caramel Oven, and Packaging Line.
* **Blacklist**: The inputs `HELPHELPHELPHELP`, `HELP`, and all-zero strings are explicitly blocked.

### Phase 2: Input Parsing & Guards
The validation function at `0x140002870` processes the input before the core logic begins.
* **Normalization**: Dashes and spaces are stripped from the input.
* **Length Validation**: The stripped string must be exactly 16 characters.
* **Timing Gate**: `GetTickCount()` is used to ensure the solution is entered within 120 seconds of program start.
* **Easter Egg**: Inputting `WONKWONKWONKWONK` triggers a hint message but does not solve the challenge.

### Phase 3: Validation Stations

#### 1. Cocoa Plantation (S-Box Substitution)
This stage validates Group 1 (`ticket[0..3]`).
* **Mechanism**: Each byte is passed through a 256-byte S-Box lookup table.
* **Anti-Debug**: An XOR mask is applied based on `IsDebuggerPresent()`. Under a debugger, the mask becomes `0x42`, causing the check to fail.
* **Target**: The result must match the constant `0xC3811DEB`.
* **Result**: The unique solution is **"Ch0c"**.

#### 2. Milk River (Dot-Product mod 256)
This stage validates Group 2 (`ticket[4..7]`).
* **Mechanism**: Uses SSE4.1 `PMULLD` to perform a dot-product between input bytes and a coefficient table.
* **Logic**: Solves four simultaneous linear equations over $Z/256Z$.
* **Result**: The only printable ASCII solution is **"M1lk"**.

#### 3. Caramel Oven (Hash Transform)
This stage validates Group 3 (`ticket[8..11]`).
* **Mechanism**: A complex 16-bit transform involving `rotl16`, XOR mixing, and byte swapping.
* **Target**: The final hash must match `0x016CB7CB`.
* **Result**: The unique solution is **"CrMe"**.

#### 4. Packaging Line (CRC-16/CCITT)
This stage validates Group 4 (`ticket[12..15]`).
* **Mechanism**: Computes a CRC-16/CCITT (poly `0x1021`) seeded by the XOR sums of the first three groups.
* **Requirement**: The final CRC residual must be `0x0000`.
* **Result**: Multiple solutions exist; a valid one is **"!(L>"**.

---

## Conclusion
The challenge is designed such that the algorithm constants reveal the intended solution ("Chocolate", "Milk", "Creme"). While the first three groups are constants, Group 4 can be solved for any valid CRC preimage.

**Tools Utilized**: `objdump`, `Python 3`, and Static Analysis.

---
**Flag**: `Ch0c-M1lk-CrMe-!(L>` 
