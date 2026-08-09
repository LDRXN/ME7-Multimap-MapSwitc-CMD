# 🔀 Bosch ME7.5 MultiMap / MapSwitch CMD

Open-source command-line tool for creating **MultiMap / MapSwitch firmware for Bosch ME7.5 1MB ECUs**.

The application automatically analyzes the ECU binary, detects the required calibration maps, creates additional calibration banks, installs the MultiMap runtime and map selectors, and generates an **XDF definition** for editing all created banks in TunerPro.

---

## ✨ Features

- 🔎 Automatic detection of required calibration maps
- 🧠 Automatic MultiMap structure generation
- 🗺️ Supports **2–4 calibration banks**
- 🔀 Runtime map switching
- 🎛️ Two selectable switching methods
- 💡 MIL / Check Engine confirmation of selected bank
- 📋 Automatic bank layout generation
- 🧩 Automatic selector installation
- 💾 Automatic output BIN creation
- 📄 Automatic CSV build report
- 🗺️ Automatic XDF generation for all MultiMap banks
- 🛡️ Original BIN is never overwritten
- ⚙️ Designed for **Bosch ME7.5 1MB**
- 🖥️ Simple interactive command-line interface
- 🔓 Open Source

---

# 🗺️ MultiMap Calibration Maps

The builder automatically searches for the calibrations required by the MultiMap system:

| Map | Description |
|---|---|
| **LAMFA** | Lambda enrichment |
| **KFZW** | Ignition timing |
| **KFZW2** | Secondary ignition timing |
| **LDRXN** | Maximum specified load |
| **LDRXNZK** | Maximum specified load – knock related |
| **KRKTE** | Injector constant |

Required axes and associated calibration data are handled automatically.

---

# 🏦 Calibration Banks

The user can create:

- **2 Banks**
- **3 Banks**
- **4 Banks**

**Bank 1 always contains the original / stock ECU calibration.**

Additional banks are automatically copied to available calibration space.

Example for a 4-bank build:

```text
Bank 1  → Original / Stock calibration
Bank 2  → MultiMap calibration 2
Bank 3  → MultiMap calibration 3
Bank 4  → MultiMap calibration 4
```

Each additional bank receives its own calibration set:

```text
Bank 2
├── LAMFA
├── KFZW
├── KFZW2
├── LDRXN
├── LDRXNZK
└── KRKTE

Bank 3
├── LAMFA
├── KFZW
├── KFZW2
├── LDRXN
├── LDRXNZK
└── KRKTE

Bank 4
├── LAMFA
├── KFZW
├── KFZW2
├── LDRXN
├── LDRXNZK
└── KRKTE
```

---

# 🎛️ Map Switching

Two switching methods are available.

## 1️⃣ Clutch + Cruise Control SET / RES

Hold the **clutch pedal** and use the cruise-control buttons:

```text
Clutch + SET  → Next Bank
Clutch + RES  → Previous Bank
```

Example with four banks:

```text
SET:

Bank 1 → Bank 2 → Bank 3 → Bank 4 → Bank 1

RES:

Bank 4 → Bank 3 → Bank 2 → Bank 1 → Bank 4
```

This allows switching in both directions through the available calibration banks.

---

## 2️⃣ Clutch + 5x Brake

Hold the **clutch pedal** and press/release the brake pedal **5 times**.

```text
Clutch + 5x Brake → Next Bank
```

There is no time limit between brake presses.

Example with four banks:

```text
Bank 1 → Bank 2 → Bank 3 → Bank 4 → Bank 1
```

---

# 💡 MIL Bank Confirmation

After a successful bank change, the **MIL / Check Engine light** confirms the currently selected calibration bank.

```text
Bank 1 → 1 blink
Bank 2 → 2 blinks
Bank 3 → 3 blinks
Bank 4 → 4 blinks
```

This makes it possible to identify the active calibration bank without any additional display or external hardware.

---

# 🔌 ECU Inputs / Outputs

Depending on the selected switching method, the MultiMap runtime uses the following ECU signals:

```text
Clutch
Brake
Cruise SET
Cruise RES
MIL / Check Engine
```

The required signals are detected and validated using the corresponding `.ecu` definition file.

Only the signals required by the selected switching method are used.

---

# 📄 ECU Definition File — `.ecu`

A matching **`.ecu` file is required**.

The `.ecu` definition can be generated from the original ECU BIN using:

```text
ME7Info.exe
```

## Generate the `.ecu` file

For example, if your original firmware is:

```text
dump.bin
```

run:

```bash
ME7Info.exe dump.bin
```

This generates:

```text
dump.ecu
```

The BIN and ECU files must use the same base filename:

```text
dump.bin
dump.ecu
```

Both files must be located in the same directory as the MultiMap builder.

Example:

```text
ME7-Multimap-MapSwitch-CMD/
│
├── MapSwitch_ME7.exe
├── ME7Info.exe
│
├── dump.bin
└── dump.ecu
```

The program automatically finds the matching `.ecu` file.

### ⚠️ Important

You do **NOT** pass the `.ecu` file on the command line.

Correct:

```bash
MapSwitch_ME7.exe dump.bin
```

Incorrect:

```bash
MapSwitch_ME7.exe dump.bin dump.ecu
```

The program automatically searches for:

```text
dump.ecu
```

when:

```text
dump.bin
```

is loaded.

---

# 📦 Required Files

Before running the builder, your working directory should contain:

```text
MapSwitch_ME7.exe
ME7Info.exe
dump.bin
dump.ecu
```

Example directory:

```text
ME7-Multimap-MapSwitch-CMD/
│
├── MapSwitch_ME7.exe
├── ME7Info.exe
├── dump.bin
└── dump.ecu
```

`ME7Info.exe` is required to generate the `.ecu` definition if you do not already have one.

Once the `.ecu` file has been generated, the MultiMap builder automatically detects it by matching the BIN filename.

---

# 🚀 Usage

## Step 1 — Prepare the original BIN

Copy your original Bosch ME7.5 1MB firmware into the program directory.

Example:

```text
dump.bin
```

---

## Step 2 — Generate the ECU definition

Run:

```bash
ME7Info.exe dump.bin
```

After successful analysis you should have:

```text
dump.bin
dump.ecu
```

---

## Step 3 — Start MultiMap Builder

Run:

```bash
MapSwitch_ME7.exe dump.bin
```

That's it.

Do not add the `.ecu` filename to the command.

The application automatically searches for the matching:

```text
dump.ecu
```

---

# 🖥️ Interactive Configuration

After loading the firmware, the program automatically:

```text
✓ Loads the BIN
✓ Finds the matching ECU definition
✓ Searches for required MultiMap maps
✓ Validates calibration addresses
✓ Validates ECU access points
```

You can then select the number of calibration banks.

Example:

```text
NUMBER OF MAP BANKS

  [1] 2 Banks
  [2] 3 Banks
  [3] 4 Banks
```

Then select the switching method:

```text
SWITCH METHOD

  [1] Clutch + Cruise SET / RES
  [2] Clutch + 5x Brake
```

The remaining MultiMap build process is performed automatically.

---

# 🗺️ Automatic XDF Generation

After a successful build, the program automatically generates an **XDF definition** for the new MultiMap firmware.

The XDF contains the original maps from **Bank 1** together with the corresponding maps from all additional calibration banks.

Example for four banks:

```text
LAMFA
LAMFA map_2
LAMFA map_3
LAMFA map_4

KFZW
KFZW map_2
KFZW map_3
KFZW map_4

KFZW2
KFZW2 map_2
KFZW2 map_3
KFZW2 map_4

LDRXN
LDRXN map_2
LDRXN map_3
LDRXN map_4

LDRXNZK
LDRXNZK map_2
LDRXNZK map_3
LDRXNZK map_4

KRKTE
KRKTE map_2
KRKTE map_3
KRKTE map_4
```

The generated XDF includes the required:

- map addresses
- bank addresses
- data types
- map dimensions
- conversion factors
- units
- axes
- axis addresses

This allows the generated MultiMap BIN to be opened directly in **TunerPro** and each calibration bank to be edited independently.

---

# 📊 Bank Layout

The program automatically calculates the memory layout for every additional calibration bank.

Example:

```text
Stock bank              : Bank 1 (original calibration)
Additional banks        : 3
Bytes / copied bank     : 704
Total copied bytes      : 2112

Bank 2   0x0C0000-0x0C02BF
  LAMFA.Y+HDR         0x01C93C -> 0x0C0000    17 B
  LAMFA.X             0x01C94E -> 0x0C0012    12 B
  LAMFA               0x01C95A -> 0x0C001E    90 B
  KFZW.X              0x0130C4 -> 0x0C0078    24 B
  KFZW.Y              0x013072 -> 0x0C0090    32 B
  KFZW                0x0120DD -> 0x0C00B0   192 B
  KFZW2               0x01219D -> 0x0C0170   192 B
  LDRXN.BLOCK         0x01EAB0 -> 0x0C0230    66 B
  LDRXNZK.BLOCK       0x01EAF2 -> 0x0C0272    66 B
  KRKTE               0x01C830 -> 0x0C02B4     2 B

Bank 3   0x0C02C0-0x0C057F
  LAMFA.Y+HDR         0x01C93C -> 0x0C02C0    17 B
  LAMFA.X             0x01C94E -> 0x0C02D2    12 B
  LAMFA               0x01C95A -> 0x0C02DE    90 B
  KFZW.X              0x0130C4 -> 0x0C0338    24 B
  KFZW.Y              0x013072 -> 0x0C0350    32 B
  KFZW                0x0120DD -> 0x0C0370   192 B
  KFZW2               0x01219D -> 0x0C0430   192 B
  LDRXN.BLOCK         0x01EAB0 -> 0x0C04F0    66 B
  LDRXNZK.BLOCK       0x01EAF2 -> 0x0C0532    66 B
  KRKTE               0x01C830 -> 0x0C0574     2 B

Bank 4   0x0C0580-0x0C083F
  LAMFA.Y+HDR         0x01C93C -> 0x0C0580    17 B
  LAMFA.X             0x01C94E -> 0x0C0592    12 B
  LAMFA               0x01C95A -> 0x0C059E    90 B
  KFZW.X              0x0130C4 -> 0x0C05F8    24 B
  KFZW.Y              0x013072 -> 0x0C0610    32 B
  KFZW                0x0120DD -> 0x0C0630   192 B
  KFZW2               0x01219D -> 0x0C06F0   192 B
  LDRXN.BLOCK         0x01EAB0 -> 0x0C07B0    66 B
  LDRXNZK.BLOCK       0x01EAF2 -> 0x0C07F2    66 B
  KRKTE               0x01C830 -> 0x0C0834     2 B
```

> **Note:** These addresses are only an example from one ECU software version.  
> The actual addresses are detected and calculated automatically for the loaded firmware.

---

# 📄 CSV Build Report

Every successful build generates a detailed CSV report.

The report contains the important information required to document the generated MultiMap firmware, including:

- selected number of banks
- selected switching method
- ECU input/output signals
- original calibration addresses
- Bank 1 calibration information
- Bank 2 calibration addresses
- Bank 3 calibration addresses
- Bank 4 calibration addresses when enabled
- source addresses
- destination addresses
- copied block sizes
- runtime information
- memory layout
- MultiMap build result

This makes it easier to verify and document exactly how a particular MultiMap BIN was generated.

---

# 📦 Output Files

For example, starting with:

```text
dump.bin
dump.ecu
```

a successful four-bank build can generate:

```text
dump_MULTIMAP_4BANK.bin
dump_MULTIMAP_4BANK.xdf
dump_MULTIMAP_4BANK_log.csv
```

The original:

```text
dump.bin
```

is never overwritten.

---

# 🔧 Editing MultiMap Banks

Open the generated BIN:

```text
dump_MULTIMAP_4BANK.bin
```

with the generated:

```text
dump_MULTIMAP_4BANK.xdf
```

in **TunerPro**.

The XDF exposes each calibration bank separately.

For example, you can prepare:

```text
Bank 1 → Stock
Bank 2 → Stage 1
Bank 3 → Stage 2
Bank 4 → Custom / Race
```

The calibration strategy itself is entirely up to the tuner.

---

# 🔄 Complete Workflow

The complete process is:

```text
┌──────────────────────────────┐
│       Original ME7.5 BIN     │
│           dump.bin           │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│          ME7Info.exe         │
│                              │
│   ME7Info.exe dump.bin       │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│          dump.ecu            │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│       MapSwitch_ME7.exe      │
│                              │
│ MapSwitch_ME7.exe dump.bin   │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│      MultiMap Build          │
│                              │
│  • Map detection             │
│  • Bank creation             │
│  • Runtime installation      │
│  • Selector installation     │
│  • XDF generation            │
│  • CSV report                │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│      MultiMap Output         │
│                              │
│  MULTIMAP.bin                │
│  MULTIMAP.xdf                │
│  MULTIMAP_log.csv            │
└──────────────────────────────┘
```

---

# ⚠️ Important

This software modifies ECU firmware.

It is intended for:

- 🔬 ECU research
- 🧪 development
- 🔧 calibration work
- 🖥️ bench testing
- 🏁 experienced ECU tuners

Always keep an untouched backup of the original firmware.

A successfully generated BIN does **not** automatically guarantee compatibility with every Bosch ME7.5 software version.

Always verify the generated firmware and ECU checksum before flashing.

**Bench testing is strongly recommended before use in a vehicle.**

Use this software at your own risk.

---

# 🎯 Supported ECU

Current target:

```text
Bosch ME7.5
1 MB Flash
C167 architecture
```

The project is currently focused specifically on **Bosch ME7.5 1MB** firmware.

---

# 🛠️ Project Status

The project is under active development.

Current development focuses on:

- 🔎 improving automatic map detection
- 🧩 improving compatibility between ME7.5 software versions
- 🔀 improving MultiMap selector generation
- 🧠 improving runtime validation
- 🗺️ improving automatic XDF generation
- 🛡️ improving safety checks before firmware creation

Testing on additional Bosch ME7.5 binaries is welcome.

---

# 🤝 Contributions

Testing, bug reports and improvements are welcome.

If you test the builder on another Bosch ME7.5 software version, please include the relevant ECU hardware/software information when reporting an issue.

---

# 📜 License

Open Source.

See the repository license for details.

---

# 🔀 Bosch ME7.5 MultiMap / MapSwitch CMD

### Automatic Map Detection • 2–4 Banks • Cruise / Brake Switching • MIL Confirmation • Automatic XDF Generation

**Build MultiMap. Edit every bank. Switch maps directly from the vehicle controls.**
