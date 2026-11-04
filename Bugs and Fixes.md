# 🔧 Bosch ME7.5 MultiMap — Core ECU Fix History (Without Pops & Bangs)

**🐞 Known Issues | 🔍 Root Causes | 🛠️ Implemented Fixes | ✅ Resolved Bugs | 🧪 Verification Status**

Technical documentation of the problems I encountered while developing Core MultiMap support for Audi A4 1.8T BFB ECU software.

> [!IMPORTANT]
> 🔥 **Pops & Bangs is supported only for the 150 / 180 / 210 / 225 HP engine families.**
>
> The ECUs documented here belong to the **163 HP family**, which is outside that range.
>
> Missing Pops & Bangs calibration maps in these ECUs is **not considered a bug**.

---

## 📊 ECU Compatibility Overview

| ECU Part Number | Engine | Status |
|---|---|---|
| `8E0909518AL` | 163 HP | 🟡 Manual fallback implemented |
| `8E0909518AF` | 163 HP | 🟡 Manual fallback implemented |
| `8E0909518AQ` | 163 HP | ✅ Patch generation PASS |

### ✍️ Manual Map Address Override

If automatic map detection fails, I use manually supplied Bank 1 calibration addresses.

**This is the intended permanent solution.**

I will not force additional automatic map detection for every software version.

However, the patcher must still validate map dimensions, axes, lookup instructions, memory locations and selector references.

---

# 🟡 1. 8E0909518AL — 163 HP

**🟡 Status: MANUAL FALLBACK AVAILABLE — FULL BUILD VALIDATION PENDING**

### 🔍 ECU Identification

| Parameter | Value |
|---|---|
| Part Number | `8E0909518AL` |
| Bosch HW | `0261208228` |
| Bosch SW | `1037369311` |
| Software Version | `0003` |

### 🐞 Problem — Required Calibration Maps Not Found Automatically

**🚨 Original symptoms**

The automatic map detector could not validate all required Core MultiMap calibration maps.

The patching process could not safely continue without the correct original addresses.

### ❓ What Caused the Problem?

The existing map-search patterns did not cover every calibration and lookup structure in this ECU software.

This did not necessarily mean that the maps were missing.

It meant that the automatic detector could not reliably identify them.

### 🛠️ How I Addressed It

I introduced the **Manual Map Address Override** mechanism.

The patcher can now accept the original Bank 1 payload addresses entered by the user.

The validation process checks the supplied addresses before continuing.

### ✅ What Has Been Resolved?

- ✅ Manual map-address entry is supported.
- ✅ Automatic detection is no longer mandatory when valid manual addresses are available.
- ✅ The decision to keep manual entry as the fallback is final.

### 🔬 What Still Requires Verification?

- 🔬 Correct locations of all six required Core maps.
- 🔬 Map axes and code lookup references.
- 🔬 Selector installation.
- 🔬 Complete 2-, 3- and 4-bank BIN/XDF generation.

### 🟡 Final Result

The automatic-map-detection limitation has an accepted solution.

**A complete successful patch-generation test has not yet been confirmed for this ECU.**

---

# 🟡 2. 8E0909518AF — 163 HP

**🟡 Status: MANUAL FALLBACK AVAILABLE — COMPLETE VALIDATION PENDING**

### 🔍 ECU Identification

| Parameter | Value |
|---|---|
| Part Number | `8E0909518AF` |
| Bosch HW | `0261207939` |
| Bosch SW | `1037366883` |
| Software Version | `0003` |

### 🐞 Problem — Automatic Calibration Detection Limitations

The patcher did not reliably recognize some required Core MultiMap calibrations in this firmware.

Previously, an automatic-detection failure could prevent the program from continuing even when the correct map addresses were known.

### ❓ What Caused the Problem?

Different ME7.5 firmware versions can use different calibration layouts and C167 lookup structures.

A map-search signature that works in one ECU is not guaranteed to work in another.

### 🛠️ How I Addressed It

I added manual Bank 1 map-address entry and validation.

The supplied address is checked against the BIN and the expected calibration structure.

The patcher must also resolve the corresponding axes and C167 lookup references before installing selectors.

### ✅ What Has Been Resolved?

- ✅ Manual map-address entry is available.
- ✅ The automatic-map-detection limitation has an accepted workaround.
- ✅ No further automatic-signature development is planned solely for this ECU.

### 🔬 What Still Requires Verification?

- 🔬 Complete validation of the required Core maps.
- 🔬 Selector references and hook installation.
- 🔬 Generation of valid 2-, 3- and 4-bank output.
- 🔬 Static verification of the resulting firmware.

**No separate Tach CAN failure has been confirmed for AF.**

### 🟡 Final Result

Manual map entry solves the original workflow limitation.

Full ECU compatibility remains unconfirmed until the generated output has been checked.

---

# 🟢 3. 8E0909518AQ — 163 HP

**✅ Status: PATCH GENERATION PASS — CODE VERIFICATION PENDING**

### 🔍 ECU Identification

| Parameter | Value |
|---|---|
| Part Number | `8E0909518AQ` |
| Bosch HW | `0261208524` |
| Bosch SW | `1037372505` |
| Software Version | `0010` |
| ECU Flash | 1 MB |

AQ required several corrections before the patcher could successfully complete the MultiMap build.

## 🐞 Problem 1 — KFZW and KFZW2 Not Found

### 🚨 Original Problem

Automatic detection could not validate both ignition maps.

Even after supplying their addresses manually, the patcher initially failed to resolve their shared axes.

### ❓ What Caused the Problem?

AQ uses a different C167 lookup structure and an alternative Bosch helper.

The original axis-detection logic did not cover this layout.

### 🛠️ How I Fixed It

I extended axis recovery to use the actual ignition lookup instructions while keeping manual map-address entry.

**📍 Verified Addresses**

| Component | Address |
|---|---|
| KFZW | `0x012850` |
| KFZW2 | `0x012910` |
| Shared X Axis | `0x01398C` |
| Shared Y Axis | `0x013906` |

**✅ Result:** Both ignition maps and their shared axes pass patch-generation validation.

---

## 🐞 Problem 2 — Five Ignition Lookup Calls

### 🚨 Original Problem

The earlier selector logic did not support all the ignition lookup locations in AQ.

### ❓ What Caused the Problem?

This ECU contains five relevant ignition calls using an alternative Bosch call instruction:

```text
DA 00 24 2B
```

### 🛠️ How I Fixed It

I added support for all five lookup locations.

| Lookup | Address |
|---|---|
| KFZW Call 1 | `0x0B8558` |
| KFZW2 Call 1 | `0x0B8586` |
| KFZW Call 2 | `0x0B85B0` |
| KFZW2 Call 2 | `0x0B85CE` |
| KFZW Call 3 | `0x0B85F8` |

Original Bosch helper:

```text
0x0344D0
```

**✅ Result:** All five ignition call sites are included in successful patch generation.

---

## 🐞 Problem 3 — LDRXN and LDRXNZK

### 🚨 Original Problem

The patcher could not automatically resolve the required LDR calibration maps and their lookup references.

### 🛠️ How I Fixed It

I identified the correct map payload locations and C167 lookup calls.

| Component | Address |
|---|---|
| LDRXN | `0x029C8A` |
| LDRXNZK | `0x029CCC` |
| LDRXN Lookup | `0x08BEA2` |
| LDRXNZK Lookup | `0x08BE88` |

### ⚠️ Important — Manual Address Correction

During testing, the following error appeared:

```text
LDRXN: manual address 0x029C68 is not
preceded by a 16-point WORD axis header
```

The address entered was the beginning of the complete map block, not the actual calibration payload.

| Map | Incorrect Address | Correct Address |
|---|---|---|
| LDRXN | `0x029C68` | `0x029C8A` |
| LDRXNZK | `0x029CAA` | `0x029CCC` |

The correct payload begins **0x22 bytes** after the block start.

This was an incorrect manual address, not a new firmware patching bug.

**✅ Result:** Manual LDR validation succeeds with the corrected addresses.

---

## 🐞 Problem 4 — Tachometer CAN Hook Not Found

### 🚨 Original Error

```text
Tach CAN hook: NOT FOUND
nmot_w loads: 58
validated CAN candidates: 0
structural RPM packers: 0
```

### ❓ What Caused the Problem?

The original Tach CAN detector expected a direct transfer from `nmot_w` to register `R4`.

However, AQ uses an additional register.

### 🔍 Original C167 Instructions

```text
0x045076  F2 F9 9E F8   MOV  R9, nmot_w
0x04507A  F0 49         MOV  R4, R9
0x04507C  F7 F8 24 F8   MOVB F824, RL4
...
0x045086  7C 84         SHR  R4, #8
0x045088  F7 F8 25 F8   MOVB F825, RL4
```

The actual RPM processing sequence is:

```text
nmot_w
   |
   v
Register R9
   |
   v
Register R4
   |
   v
F824 / F825
   |
   v
CAN RPM data
```

### 🛠️ How I Fixed It

I improved the Tach CAN detector to recognize this alternative register sequence.

The changes included:

- ✅ Detecting the `nmot_w → R9 → R4` structure.
- ✅ Validating the complete RPM-packing sequence.
- ✅ Avoiding fixed ECU-specific hook addresses.
- ✅ Updating the wrapper to use the correct register path.
- ✅ Preserving the previous direct-R4 detection mechanism.

### 📍 Tach CAN Locations

| Component | Address |
|---|---|
| Tach CAN Hook | `0x045076` |
| Tach Wrapper — tested 4-bank build | `0x0BFB60` |
| RPM Buffer | `0xF824 / 0xF825` |

### ✅ Final Result

**The Tach CAN detection problem has been resolved at the patch-generation level.**

The patcher now finds the hook and generates MultiMap output without disabling Tach CAN.

Actual tachometer behavior must still be verified.

---

## 🧠 Runtime and Memory Validation

| Component | Address |
|---|---|
| Cyclic Hook | `0x0B8944` |
| Runtime Code Cave | `0x0BF9A0` |
| Runtime RAM | `0x384420` |
| Tach CAN Hook | `0x045076` |
| Core Selector Call Sites | 11 |

The patcher retains the original Bank 1 calibration and creates additional calibration copies for the selected banks.

I did not change unrelated XDF or calibration logic solely to implement the Tach CAN fix.

---

## 🧪 AQ Verification Results

| Test | Result |
|---|---|
| Original BIN and ECU definition | ✅ PASS |
| Manual map-address validation | ✅ PASS |
| Ignition axes | ✅ PASS |
| Five ignition selectors | ✅ Build PASS |
| LDR lookup references | ✅ PASS |
| Cyclic hook / runtime allocation | ✅ PASS |
| Tach CAN detection | ✅ PASS |
| 2-bank generation | ✅ PASS |
| 3-bank generation | ✅ PASS |
| 4-bank generation | ✅ PASS |
| Full C167 disassembly verification | 🔬 PENDING |
| Real ECU / vehicle testing | 🔬 NOT TESTED |

**✅ AQ patch generation is now working correctly according to the completed build checks.**

---

# 🧩 Summary of Implemented Fixes

| Component | Improvement | Status |
|---|---|---|
| 🗺️ Map Detection | Manual address entry as permanent fallback | ✅ Implemented |
| 📈 Ignition Axes | Alternative C167 lookup handling | ✅ Fixed in AQ |
| 🔗 KFZW/KFZW2 | Support for five ignition lookup calls | ✅ Fixed in AQ |
| 📉 LDR Maps | Correct payload and lookup identification | ✅ Fixed in AQ |
| 📡 Tach CAN | Support for the R9-to-R4 RPM path | ✅ Fixed in AQ build |
| 🧠 Runtime Allocation | Validated runtime and memory locations | ✅ Build PASS in AQ |
| 🧪 AL / AF | Full Core build verification | 🔬 Pending |
| 🔍 C167 Runtime Validation | Independent disassembly comparison | 🔬 Pending |

---

# 🔒 Development Policy

### ✅ What I Will Keep

- Manual map-address entry when automatic detection fails.
- Existing map and selector validation.
- Working memory-allocation and runtime mechanisms.
- The corrected AQ ignition and LDR lookup handling.
- The new Tach CAN detection mechanism.
- Existing XDF behavior unless a reproducible issue is found.

### 🚫 What I Will Not Change Unnecessarily

- I will not force automatic map detection for every ECU software version.
- I will not add Pops & Bangs support to unsupported 163 HP applications.
- I will not modify working components without evidence of a defect.
- I will not consider successful patch generation proof of safe vehicle operation.

---

# 📋 Remaining Work

| ECU | Remaining Task |
|---|---|
| `8E0909518AL` | Confirm complete Core MultiMap generation using correct manual map addresses |
| `8E0909518AF` | Validate full BIN/XDF generation and selector references |
| `8E0909518AQ` | Independently verify generated C167 code through disassembly |

> [!WARNING]
> ⚠️ **BUILD PASS DOES NOT MEAN VEHICLE TESTED.**
>
> Successful patch generation confirms that the patcher completed its documented build checks.
>
> Runtime correctness, checksum validation, ECU compatibility and safe vehicle operation must be verified separately.

---

# 🔬 Final Verification — Next Development Stage

**The AQ patcher now works at the patch-generation stage.**

It successfully accepts the correct manually supplied calibration addresses, installs the required selectors and Tach CAN hook, and generates BIN/XDF output for 2, 3 and 4 banks.

**My next step will be to disassemble the original AQ firmware and the modified firmware.**

I will compare the generated C167 instructions and verify:

- 🔍 Original and overwritten instructions.
- 🧩 All installed hooks and selectors.
- 🧠 Register preservation and restoration.
- 🔗 Branch and return behavior.
- 💾 RAM and free-space allocation.
- 📈 Calibration-bank references.
- 📡 Tach CAN wrapper and RPM processing.
- ✅ Correctness of the generated runtime logic.

Only after this independent disassembly and comparison will I consider the code-level implementation verified.

**For AL and AF, manual map-address entry remains the intended solution. Full patch-generation validation is still pending.**

Actual ECU and vehicle testing will be a separate final stage.

---

**Developed by LDRXN**

[GitHub — LDRXN](https://github.com/LDRXN)
