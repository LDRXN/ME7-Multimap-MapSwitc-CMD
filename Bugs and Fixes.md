# 🔧 Bosch ME7.5 MultiMap — Core ECU Fix History (Without Pops & Bangs)

![ECU](https://img.shields.io/badge/ECU-Bosch%20ME7.5-3B82F6?style=flat-square) ![Scope](https://img.shields.io/badge/Scope-Core%20MultiMap-64748B?style=flat-square) ![ECUs](https://img.shields.io/badge/ECUs-3-2563EB?style=flat-square) ![AL](https://img.shields.io/badge/AL-Build%20PASS-16A34A?style=flat-square) ![AQ](https://img.shields.io/badge/AQ-Build%20PASS-16A34A?style=flat-square) ![Testing](https://img.shields.io/badge/C167%20Validation-Pending-F59E0B?style=flat-square)

**🐞 Original Bugs | 🔍 Root Causes | 🛠️ Implemented Fixes | ✅ Results | 🧪 Remaining Verification**

This is the technical fix history for **Core MultiMap**, covering three Bosch ME7.5 ECU software variants associated with the **Audi A4 1.8T BFB 163 HP (120 kW)** engine family.

**ECUs covered:** `8E0909518AL` · `8E0909518AF` · `8E0909518AQ`

> [!IMPORTANT]
> 🔥 **Pops & Bangs is not part of this report.** My project's Pops & Bangs support is limited to the **150 / 180 / 210 / 225 HP** engine-power families. These investigated **163 HP** variants are outside that range. Missing Pops-specific maps in these ECUs is **expected behavior, not a bug**, and does not require a fix.

> [!NOTE]
> ✍️ **Manual Map Address Override is the final, intended fallback.** When a required Core map cannot be located automatically, I enter its correct **Bank 1 payload address** manually. I do not plan to force additional automatic map signatures for every ECU version. The patcher must still validate map structure, axes, C167 lookup references, hooks and memory allocation.

---

## 📊 ECU Compatibility & Fix Overview

| ECU part number | Bosch HW | Bosch SW | SW version | Factory application | Status | Remaining work |
|---|---|---|---|---|---|---|
| **`8E0909518AL`** | `0261208228` | `1037369311` | `0003` | 163 HP | ✅ **Build PASS — 2/3/4 banks** | Independent C167 disassembly and physical ECU testing pending |
| **`8E0909518AF`** | `0261207939` | `1037366883` | `0003` | 163 HP | 🟡 **Manual fallback implemented** | Validate actual map references and full BIN/XDF generation |
| **`8E0909518AQ`** | `0261208524` | `1037372505` | `0010` | 163 HP | ✅ **Patch generation PASS** | Independently disassemble and verify generated C167 code; hardware testing remains pending |

**Status definitions:**

- ✅ **Build PASS:** Successful BIN/XDF generation for the bank counts stated, including the patcher's pre-flight checks. It does **not** prove correct runtime behavior or in-vehicle operation.
- 🟡 **Manual fallback available / verification pending:** Map entry is possible, but an end-to-end successful build for that particular ECU has not yet been confirmed.
- 🔬 **Code-level validation pending:** Original-versus-modified C167 disassembly, runtime auditing and real ECU testing are separate stages.

### 🧩 Core MultiMap Scope

The Core builder handles **six calibration maps**:

| Fueling | Ignition | Load / Boost |
|---|---|---|
| `LAMFA`, `KRKTE` | `KFZW`, `KFZW2` | `LDRXN`, `LDRXNZK` |

**Bank 1** retains its original calibration locations. Additional banks receive separate map copies, with validated C167 lookup selectors used to choose the active bank.

The builder also prepares its switching runtime, optional tachometer bank indication, and matching XDF definitions.

---

# 🟢 1. 8E0909518AL — Audi A4 1.8T BFB · 163 HP

**✅ Current Status: BUILD PASS — 2 / 3 / 4 BANKS · C167 DISASSEMBLY PENDING**

### 🔍 ECU Identification

| Parameter | Value |
|---|---|
| ECU Part Number | `8E0909518AL` |
| Bosch HW | `0261208228` |
| Bosch SW | `1037369311` |
| Software Version | `0003` |
| Factory Engine Application | Audi A4 1.8T BFB, 163 HP |
| Flash Size | 1 MiB |

During the AL investigation, I encountered **three connected issues**.

The final build succeeded after supplying map addresses that matched the **actual AL BIN**.

I did not need to force additional automatic-map recognition.

## 🐞 Problem 1 — Core Maps Not Found by Automatic Detection

### 🚨 Original Symptoms

Automatic search did not validate the following maps:

```text
KFZW       AUTO=NOT FOUND
KFZW2      AUTO=NOT FOUND
LDRXN      AUTO=NOT FOUND
LDRXNZK    AUTO=NOT FOUND
```

The patcher requested manual Bank 1 addresses before it could continue.

### ❓ Why It Happened

The generic map patterns did not resolve all calibrations and references for this software layout.

A missed automatic signature does **not** mean the calibration is absent.

### 🛠️ How I Addressed It

I retained **Manual Map Address Override** as the permanent fallback.

The patcher now allows me to:

- ✅ Enter the correct map addresses manually.
- ✅ Validate map geometry and calibration data.
- ✅ Resolve the required axes.
- ✅ Validate C167 lookup sites before installing selectors.
- ✅ Continue patch generation without forcing new automatic signatures.

I am **not** extending automatic detection merely to eliminate a manual entry step.

### ✅ Result

**Automatic detection is no longer a blocker when correct addresses are available.**

---

## 🐞 Problem 2 — Incorrect Map Addresses from a Multi-ECU WinOLS KP

### 🚨 Original Manual Entries

```text
KFZW       0x012120
KFZW2      0x0121E0
LDRXN      0x01F076
LDRXNZK    0x01F0B8
```

These entries resulted in the following error:

```text
KFZW: manual map address was supplied,
but the shared 12x16 KFZW axes could not
be resolved.
```

### ❓ What Caused the Problem?

The supplied WinOLS KP contained map definitions for multiple ECU software variants.

Although the addresses appeared in the KP, they were **not confirmed for the exact AL BIN**.

After inspecting the original firmware, I found that the C167 ignition references and calibration layout did not match the initially supplied addresses.

### 🛠️ How I Fixed It

I compared the original `AL.bin` with its C167 map-lookup paths and identified the addresses matching the actual firmware.

Instead of trusting the unverified KP entries, I used the locations validated against the supplied BIN.

### 📍 Final Map Addresses Used in Successful Builds

| Required Core Map | Address in Tested AL BIN | Detection |
|---|---|---|
| `LAMFA` | `0x026E66` | ✅ Automatic |
| `KFZW` | `0x01284F` | ✅ Manual |
| `KFZW2` | `0x01290F` | ✅ Manual |
| `LDRXN` | `0x029C8A` | ✅ Manual |
| `LDRXNZK` | `0x029CCC` | ✅ Manual |
| `KRKTE` | `0x01FA14` | ✅ Automatic |

> [!NOTE]
> The KFZW/KFZW2 values above are the **exact addresses accepted and copied by the tested builder** (`0x01284F` and `0x01290F`).
>
> Do not substitute adjacent addresses from another BIN or definition without checking the actual code and map representation.

### ✅ Result

All six required Core maps were successfully validated.

```text
Core Maps: 6/6 VALIDATED
```

The patcher advanced to runtime and selector installation.

---

## 🐞 Problem 3 — Manual KFZW/KFZW2 Axes Failed Validation

After the initial map entries failed, I investigated the axis addresses listed in the KP.

### 🚨 Original Axis Addresses

| Axis | Address | Result |
|---|---|---|
| KFZW/KFZW2 X — 12 points | `0x01311C` | ❌ Invalid |
| KFZW/KFZW2 Y — 16 points | `0x0130CA` | ❌ Invalid |

The diagnostic confirmed that these addresses were not valid 12/16-point WORD axes in the supplied firmware.

The preceding values were not valid axis headers, and the sampled values did not form the expected monotonic sequences.

### 🚨 Original Error

```text
KFZW/KFZW2 X: 0x01311C is neither a
validated 12-point WORD axis header
nor a payload immediately after one.
```

### ❓ Why It Happened

The addresses were taken from an unverified multi-ECU definition.

The patcher's axis validation correctly rejected them because their structure did not match the expected Bosch calibration format.

### 🛠️ How I Resolved It

I inspected the actual AL firmware and followed the ignition lookup structure in C167.

This allowed the patcher to recover the correct shared axes.

### 📍 Correct KFZW/KFZW2 Axes

| Axis | Payload Address | Geometry |
|---|---|---|
| X — Load | `0x01398C` | 12 WORD values |
| Y — RPM | `0x013906` | 16 WORD values |

**No manual axis override was needed in the final successful builds.**

The correct solution was to provide valid map addresses and let the existing C167 validation recover the corresponding axes.

### ✅ Result

- ✅ Correct ignition axes identified.
- ✅ Invalid KP axis addresses rejected.
- ✅ Map geometry validated.
- ✅ Existing safety checks preserved.
- ✅ Patch generation continued successfully.

---

## 🔗 AL Selector and Runtime Results

The tested four-bank build selected **11 Core call sites**, including five ignition lookups.

### 📍 Verified Selector Locations

| Function | C167 Call / Hook Address |
|---|---|
| KFZW Call 1 | `0x0B83F8` |
| KFZW2 Call 1 | `0x0B8426` |
| KFZW Call 2 | `0x0B8450` |
| KFZW2 Call 2 | `0x0B846E` |
| KFZW Call 3 | `0x0B8498` |
| LAMFA Call | `0x076422` |
| LDRXNZK Call | `0x08BE88` |
| LDRXN Call | `0x08BEA2` |
| KRKTE R5 | `0x071CC8` |
| KRKTE R4 | `0x0B959A` |
| KRKTE R2 | `0x0B96F2` |

### 🧠 Runtime and Memory Locations

| Component | Address / Result |
|---|---|
| Cyclic Hook | `0x0B87E4` |
| Runtime RAM | `0x384420` |
| Tach CAN Hook | `0x045076` |
| RPM CAN Instruction Path | `nmot_w → R9 → R4 → F824/F825` |
| Selector Call Sites | 11 |
| Copied Calibration Blocks — 4 Banks | 30 blocks |

The four-bank build created 10 copied calibration blocks for each additional bank.

The original Bank 1 calibration was preserved.

> [!IMPORTANT]
> Successful Tach CAN hook detection confirms that the patcher located and installed the hook during generation.
>
> It does not yet prove correct tachometer behavior in the physical vehicle.

---

## 🧪 AL Verification Results

| Test | Result |
|---|---|
| Original AL BIN and ECU definition loaded | ✅ PASS |
| Six Core calibrations validated | ✅ 6/6 |
| Manual ignition and LDR map entry | ✅ PASS |
| Shared 12×16 ignition axes identified | ✅ PASS |
| Eleven selector call sites included | ✅ Build PASS |
| Cyclic Hook / Runtime RAM pre-flight | ✅ PASS |
| Tach CAN hook detection / installation | ✅ Build PASS |
| 2-bank BIN/XDF generation | ✅ PASS |
| 3-bank BIN/XDF generation | ✅ PASS |
| 4-bank BIN/XDF generation | ✅ PASS |
| Original vs modified C167 disassembly | 🔬 PENDING |
| Real ECU / Vehicle Testing | 🔬 NOT YET CONFIRMED |

Example for AL 

python3 MapSwitch_ME7_V2_V9_CORE_MANUAL_KFZW_AXES.py AL.bin \
  --addr-kfzw 0x1284F \
  --addr-kfzw2 0x1290F \
  --addr-ldrxn 0x29C8A \
  --addr-ldrxnzk 0x29CCC \
  --banks 4 \
  --switch-mode cruise




### ✅ Final AL Result

**The AL firmware now passes complete 2-, 3- and 4-bank patch generation.**

The earlier calibration-address and ignition-axis problems were resolved without forcing new automatic map-detection patterns.

The patcher now generates BIN/XDF output successfully using the addresses corresponding to the supplied original firmware.

**Full C167 disassembly and hardware validation remain pending.**

---

# 🟡 2. 8E0909518AF — Audi A4 1.8T BFB · 163 HP

**🟡 Current Status: MANUAL FALLBACK AVAILABLE — COMPLETE VALIDATION PENDING**

### 🔍 ECU Identification

| Parameter | Value |
|---|---|
| ECU Part Number | `8E0909518AF` |
| Bosch HW | `0261207939` |
| Bosch SW | `1037366883` |
| Software Version | `0003` |
| Engine Application | Audi A4 1.8T BFB, 163 HP |

## 🐞 Problem — Automatic Detection Was Not Sufficient for Every Core Map

### 🚨 Original Symptoms

Some required calibration maps were not reliably recognized in the AF firmware.

Previously, an automatic-detection failure could prevent further patch preparation even when the correct map addresses were already known.

### ❓ What Caused the Problem?

The original recognition methods depended on expected calibration and C167 lookup patterns.

Different ME7.5 software versions can use different map layouts and code-reference structures.

AF uses a different software arrangement, so successful map recognition in another ECU does not guarantee the same result here.

This issue is **not related to missing Pops & Bangs maps**.

AF belongs to the 163 HP family, which is outside the supported Pops & Bangs range in this project.

## 🛠️ How I Addressed It

I made manual Bank 1 map-address entry available whenever automatic detection cannot validate a required calibration.

The entry process checks the supplied address against the BIN and expected map structure.

### 🔧 Implemented Improvements

- ✅ Manual Bank 1 map-address entry.
- ✅ Address-range validation.
- ✅ Basic map-structure validation.
- ✅ Support for continuing after failed automatic map recognition.
- ✅ Validation of map axes and code references before selector installation.

### ⚠️ Important

A valid map payload address alone is not enough.

The patcher must also correctly resolve:

- Map dimensions.
- X/Y axes where applicable.
- C167 lookup instructions.
- Selector installation locations.
- Runtime and memory allocation.

