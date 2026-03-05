# PMV Database Editor

A Python tool for reading, editing, and creating the encrypted `.dat` database
files used by **Sigma Metalytics Precious Metal Verifier (PMV)** devices. [https://www.sigmametalytics.com]([url](https://www.sigmametalytics.com))

---

## TL;DR — Quick start

```bash
pip install pycryptodome
```

> If `pip` is not found or you get an error, see the official installation
> guide: [pip.pypa.io/en/stable/installation](https://pip.pypa.io/en/stable/installation/)

1. Download your device's latest `.dat` file from the
   [Sigma Metalytics database updater page](https://www.sigmametalytics.com/pages/database-updaters).
2. Open it:
   ```bash
   python pmv_editor.py "Invest 1.15.dat"
   ```
3. Use the menu — press `n` to add a new metal (guided wizard), `e` to edit an
   existing one, `l` to list all records.
4. Press `s` to save.  A backup of the original file is created automatically.
5. Load the saved `.dat` file onto your device using the Sigma Metalytics
   updater software as normal.

> **Important:** the `.dat` file is a full replacement — it overwrites
> everything on the device.  Always start from the official downloaded file,
> not a blank one, so the factory records are preserved.

---

Supported devices:

| Device | Database filename |
|---|---|
| PMV Investor | `Invest x.xx.dat` |
| PMV Standard | `PMV Database x.xx.dat` |
| PMV Pro | `PRO x.xx.dat` |
| PMV Mini | `PMVMiniDatabase.dat` |

All four device types use the same file format and encryption key, so this
tool works on all of them without any configuration.

---

## Disclaimer

This software is provided **as is**, without warranty of any kind.  The
authors accept no responsibility for any consequences arising from its use,
including but not limited to: incorrect database entries, device malfunction,
failed authentication of genuine metals, or acceptance of counterfeit items.

Always verify results independently before relying on any PMV device for
high-value transactions.  Use this tool at your own risk.

---

## Important: the database is a full replacement

> **Warning — loading a modified `.dat` file onto your PMV device will
> completely erase and replace the device's existing database.**

The `.dat` file is not a patch or a set of updates — it is the entire database
in one file.  When the device loads a new file, every record that was
previously stored on the device is gone and replaced by whatever is in the
new file.

**What this means in practice:**

- **Always start from an existing official database.**  Download the latest
  official `.dat` file for your device from the
  [Sigma Metalytics website](https://www.sigmametalytics.com/pages/database-updaters),
  open it with this tool, add or edit your records, and then save and load the
  result.  This way all the factory-calibrated entries are preserved alongside
  your custom ones.

- **A backup is created automatically every time you save.**  The first time
  you press `s` in a session, the tool writes a copy of the originally loaded
  file to `Backup-<original filename>` in the same directory before overwriting
  anything.  If you ever need to restore the previous state, the backup file
  will be there.

---

## Requirements

Python 3.8 or newer.

```bash
pip install pycryptodome        # required for all operations
pip install dnfile              # only needed for --extract-key
```

---

## Quick start

```bash
# Open and edit a database interactively
python pmv_editor.py MyDatabase.dat

# Extract key/IV from the Windows downloader if the key has changed
python pmv_editor.py --extract-key InvestorDatabaseDownloader.exe
```

---

## Interactive menu

When a file is opened, a text menu is shown:

```
───────────────────────────────────────────────────────────────────────────────
  DATABASE: Invest 1.15  (49 records)
───────────────────────────────────────────────────────────────────────────────
  l              List all records in the database
  v [#]          View all fields of one record in detail
  e [#]          Edit the fields of one record
  n              Add a new metal record (guided wizard)
  d [#]          Delete a record permanently
  c [#]          Duplicate a record as a starting point for a new one
  s              Save all changes back to the current file (auto-backup created)
  q              Quit — you will be warned if there are unsaved changes

  Tip: for v, e, d and c you can include the record number directly,
       e.g.  'v 5'  or  'e 12'  to skip the list step.
```

Changes are tracked in memory until you save.  You will be warned if you try
to quit with unsaved changes.

### List (`l`)

Displays every record as a single wide table with all columns visible —
record number, name, category, and all 9 measurement fields.

### View (`v [#]`)

Shows the full detail of one record as a table with field names in the header
row and values in the data row.  You can include the record number directly in
the command (e.g. `v 5`) to skip the list step.

### Edit (`e [#]`)

Steps through every field of a record one at a time.  Press Enter to keep the
current value, or type a new one.  You can include the record number directly
(e.g. `e 12`).

### Add new record (`n`)

A five-step guided wizard that walks you through every field needed to add a
new metal, explaining in plain language what each value means and how to find
it.  All values are entered manually.

See **[New record wizard](#new-record-wizard)** below for full instructions.

### Delete (`d [#]`)

Shows the full record detail first so you can confirm what will be removed,
then asks for confirmation before deleting.  This cannot be undone without
reloading from the original file.  You can include the record number directly
(e.g. `d 3`).

### Copy (`c [#]`)

Duplicates an existing record and inserts it immediately after the original,
named `<original name> (copy)`.  Use `e` afterwards to rename and adjust the
copy.  You can include the record number directly (e.g. `c 7`).

### Save (`s`)

Before writing anything to disk, `s` automatically creates a backup of the
**originally loaded file** named `Backup-<original filename>` in the same
directory.  The backup is only created once per session — if it already exists
it is left untouched, so it always reflects the true state of the file before
any edits were made in this or any previous session.

Example: opening `Invest 1.15.dat` and pressing `s` will produce
`Backup-Invest 1.15.dat` alongside the saved file.

---

## New record wizard

The `n` command opens a five-step guided wizard.  Each step explains what the
field means and how to find the correct value.  All values are entered manually
by the user — the wizard does not connect to the device automatically.

### Before you start

You will need:
- At least one known **genuine** sample of the metal you want to add.
  More pieces (3–5 from different sources) give a more reliable reading range.
- The **specific gravity** (density) of the metal — see the reference table in
  Step 5 below, or look up the published value for your specific alloy.
- Your PMV device, to take bar readings from the genuine sample.

### How the measurement bar works

Every PMV device displays a measurement bar when a sample is placed on it.
The bar is divided into colour-coded zones:

```
Red (fail) | Yellow (caution) | Green (pass) | Yellow (caution) | Red (fail)
```

A genuine sample should produce a reading that lands in the **green zone**.
The four database threshold values define exactly where each zone boundary sits.
The wizard asks for the range of readings you observe on genuine samples and
calculates the boundaries for you.

The device also shows a **numeric value** alongside the bar — that number is
what you enter in Step 4.

### How to read the bar value from your device

#### PMV Investor

1. Power on and wait for the home screen.
2. Place the sample flat on both probes (left and right contact pads).
3. The device runs **Basic Verification** (surface resistivity) first, then
   **Thru Verification** (bulk resistivity).
4. Each screen shows a horizontal bar and a **numeric value** below it
   (e.g. `2.32`).  Note this number.
5. Repeat with each genuine sample.  Note the lowest and highest values you see.

#### PMV Pro

1. Power on and place the sample on both probes.
2. The device shows a **Basic** bar reading, then a **Thru** bar reading.
   Each screen shows a numeric value (e.g. `1.71`).
3. Note the numeric value — not the LED colour, but the actual number shown.
4. Repeat across several genuine pieces and record the full range (min and max).

#### PMV Standard

1. Place the sample on the probes.
2. The display steps through Basic and Thru verification screens in sequence,
   each showing a bar and a numeric reading.
3. Note the reading from each screen and record the range across all your
   genuine samples.

#### PMV Mini

1. Place the sample flat on the probe surface.
2. Wait for the reading to stabilise (stops changing).
3. Note the numeric value shown on or beside the bar.
4. Repeat for each genuine sample and record the lowest and highest values.

---

### Wizard steps

Select `n` from the main menu.  The wizard proceeds through five steps.

**Step 1 — Metal name**

Enter the label exactly as you want it to appear on the device display when
this metal is tested (e.g. `.999`, `18k Yellow`, `Maple Leaf 1oz`).
Keep it short — long names may be truncated on the device screen.

**Step 2 — Category**

Choose which group this metal belongs to:

```
0 = Gold          — all gold alloys and gold-plated items
1 = Silver        — all silver alloys
2 = Other         — platinum, palladium, rhodium, copper, calibrators, etc.
3 = Coins/Bullion — specific named coins where size and weight are also checked
```

**Step 3 — Green zone left offset (ResGreenLeft)**

`ResGreenLeft` defines how far from the left edge of the measurement bar the
green (pass) zone begins.  It is stored in a different scale to the resistance
readings shown on the display (roughly 350–2200 vs. the 1–20 range of the bar
readings), so its value cannot be read directly off the device screen.

The best approach is to copy it from the most physically similar existing
record — a metal whose green zone starts at roughly the same position on the
bar.  The wizard lists all existing records grouped by category with their
`ResGreenLeft` values so you can choose.

Rules of thumb:
- New gold alloy → copy from the gold record closest in purity.
- New silver alloy → copy from the silver record closest in purity.
- New coin → copy from a coin of the same base metal.
- No similar record exists → copy the nearest match and use `e` to fine-tune
  later if the device does not respond as expected.

**Step 4 — Measurement bar thresholds**

Enter the **lowest** and **highest** bar readings you observed across all your
genuine samples.  The wizard uses these to set the green (pass) zone:

```
ResYellowLeft  = lowest  × 0.93   ← outer left boundary (below = red/fail)
ResGreenRight  = lowest            ← green zone starts here
ResYellowRight = highest           ← green zone ends here
Field5         = highest × 1.07   ← outer right boundary (above = red/fail)
```

The 7 % margin on each side creates the yellow caution zone.  A reading there
means "close to the genuine range but not firmly within it".  You can change
the margin percentage at this step if you want a wider or narrower buffer.

If you only have one genuine sample, use the same value for both lowest and
highest.  The wizard will still apply the margin to create a caution zone.

**Step 5 — Specific gravity and tolerances**

*Specific gravity* is the density of the metal relative to water (pure water
= 1.0).  Gold at 19.3 g/cm³ is about 19.3 times denser than water.  The
device uses this value in its hydrostatic weighing mode.

Common reference values:

| Metal | g/cm³ | Metal | g/cm³ |
|---|---|---|---|
| Gold | 19.30 | Platinum | 21.45 |
| Silver | 10.49 | Palladium | 12.02 |
| Tungsten | 19.25 | Rhodium | 12.41 |
| Lead | 11.34 | Copper | 8.96 |

For alloys, use the published density for that specific composition
(e.g. 18k yellow gold ≈ 15.5 g/cm³, 14k yellow gold ≈ 13.1 g/cm³).

*Dimension tolerances* (`DimensionModePlusTolerance` and
`DimensionModeMinusTolerance`) and the *weight multiplier*
(`TotalWeightMultiplier`) are only relevant in Dimension/Coin mode, where the
device checks that the physical size and weight of a sample match the record.
For standard resistivity testing these values have no effect.  The wizard
defaults them from the nearest same-category existing record; press Enter to
accept the defaults unless you are adding a coin record that needs precise
size and weight limits.

**Confirmation**

A full summary of every field is shown before the record is added.  Confirm
with `y`, then use `s` to save the database file.

---

### Tips for better results

- **Clean contacts** — wipe both the sample and the probe contacts before
  measuring.  Oxidation or debris shifts readings.
- **Consistent placement** — lay the sample flat with firm, even contact on
  both probes.  Tilted or partial contact causes unstable readings.
- **Wait for stability** — the readout may fluctuate for a second after
  placement.  Note the value only once it has settled.
- **More samples is better** — a single piece gives you one data point; five
  pieces show the natural spread of the genuine range.
- **Test with a known fake** — after loading the new database onto the device,
  test against a known counterfeit if available.  A correctly set record will
  show red or yellow for the fake and green for genuine pieces.
- **Use `c` then `e` as an alternative** — if the new metal is very similar
  to one already in the database, duplicating that record with `c` and
  adjusting only the differing fields with `e` can be faster than the wizard.
- **Save frequently** — each save automatically backs up the original file, so
  you can always recover it if something goes wrong.

---

## Command-line reference

```
python pmv_editor.py [options] [file.dat]
```

| Option | Description |
|---|---|
| *(no arguments)* | Prompts you to enter a `.dat` file path |
| `file.dat` | Opens the specified `.dat` file |
| `--extract-key exe` | Extracts the AES key and IV from the downloader `.exe` |
| `--use-key KEY IV` | Uses the given hex key and IV instead of the built-in values |

### Examples

```bash
# Edit a specific file
python pmv_editor.py "PRO 1.42.dat"

# Extract the key from the Investor downloader, then open a file
python pmv_editor.py --extract-key "InvestorDatabaseDownloader.exe" "Invest 1.15.dat"

# Manually supply a different key and IV
python pmv_editor.py --use-key 32392013ded44052ae296db75bf03377 774abcf022b64aa193d519726f0144bd MyFile.dat
```

---

## Using as a library

`pmv_editor.py` can be imported and used in other Python scripts:

```python
from pmv_editor import Database, Record, FIELD_NAMES

# Load and decrypt
db = Database.load('Invest1.15.dat')
print(db.description, db.timestamp, len(db.records))

# Inspect records
for record in db.records:
    print(record.name, record.category, record.values[5])  # values[5] = specific gravity

# Edit a record
db.records[0].name = '.9999 Pure'
db.records[0].values[5] = 19.32   # update specific gravity

# Add a new record
new = Record(
    name        = 'My Alloy',
    category_id = 0,              # 0=Gold 1=Silver 2=Other 3=Coins/Bullion
    values      = [1975.0, 2.05, 2.1, 2.4, 2.83, 19.3, 1.0, 10.0, 10.0]
)
db.records.append(new)

# Save (re-encrypts automatically)
db.save('Invest1.15_modified.dat')

# Export to CSV
import csv
with open('database.csv', 'w', newline='') as f:
    writer = csv.DictWriter(f, fieldnames=['name', 'category'] + FIELD_NAMES)
    writer.writeheader()
    writer.writerows(record.to_dict() for record in db.records)
```

---

## Database record fields

Each precious metal record contains these fields:

| Field | Description |
|---|---|
| `name` | Metal name shown on the device display, e.g. `.999` |
| `category_id` | 0 = Gold, 1 = Silver, 2 = Other, 3 = Coins/Bullion |
| `ResGreenLeft` | Offset from the left edge of the bar to where the green zone starts (scale ~350–2200) |
| `ResYellowLeft` | Outer left boundary of the bar — below this is the red (fail) zone |
| `ResGreenRight` | Left edge of the green (pass) zone |
| `ResYellowRight` | Right edge of the green (pass) zone |
| `Field5` | Outer right boundary of the bar — above this is the red (fail) zone |
| `SpecificGravity` | Specific gravity of the metal in g/cm³ |
| `DimensionModePlusTolerance` | Upper size tolerance for dimension mode |
| `DimensionModeMinusTolerance` | Lower size tolerance for dimension mode |
| `TotalWeightMultiplier` | Multiplier applied to the total weight reading |

The four bar thresholds define the zone layout as follows:

```
  Red (fail)  |  Yellow (caution)  |  Green (pass)  |  Yellow (caution)  |  Red (fail)
              ↑                    ↑                 ↑                    ↑
        ResYellowLeft        ResGreenRight     ResYellowRight           Field5

  ResGreenLeft — separate value: offset from the left edge of the bar
                 to the start of the green zone (different scale, ~350–2200)
```

---

## File format details

### Encryption

Each `.dat` file is encrypted with **AES-128-CBC** using PKCS7 padding.

```
Algorithm : AES-128-CBC
Key size  : 128 bits (16 bytes)
Block size : 16 bytes
Padding   : PKCS7
Key       : 32392013ded44052ae296db75bf03377
IV        : 774abcf022b64aa193d519726f0144bd
```

### Binary layout (after decryption)

| Bytes | Type | Content |
|---|---|---|
| variable | LEB128 string | Database description (e.g. `Invest 1.15`) |
| variable | LEB128 string | Creation timestamp (e.g. `2/24/2026 8:57:13 AM`) |
| 4 | int32 LE | Number of records |
| *repeated per record:* | | |
| variable | LEB128 string | Metal name |
| 4 | int32 LE | Category ID |
| 8 × 9 | double LE | The 9 measurement fields listed above |

**LEB128 strings** follow the C# `BinaryWriter` convention: the string's UTF-8
byte length is stored as a variable-length prefix where each byte contributes
7 bits, and the high bit signals whether another byte follows.

---

## If the encryption key changes

### Recognising the problem

If the key has changed, the tool will fail to open any `.dat` file and display
the following error:

```
  ERROR: Failed to decrypt or parse the database file.

  The most likely cause is that Sigma Metalytics has released a new
  version of their downloader with a different AES encryption key.

  To fix this:
    1. Download the latest installer from:
       https://www.sigmametalytics.com/pages/database-updaters
    2. Extract InvestorDatabaseDownloader.exe from the installer.
    3. Run:
         python pmv_editor.py --extract-key InvestorDatabaseDownloader.exe
       This will update pmv_key.json automatically.
    4. Then open your .dat file again.

  If the problem persists, the file may be corrupted or not a valid
  PMV database file.
```

### Updating the key

Sigma Metalytics may update the AES key in a future release of their downloader
software.  The tool handles this automatically:

1. Download the new installer from the Sigma Metalytics website.
2. Extract `InvestorDatabaseDownloader.exe` from the installer.
3. Run:
   ```bash
   python pmv_editor.py --extract-key InvestorDatabaseDownloader.exe
   ```
4. If the key has changed, the tool writes the new values to `pmv_key.json`
   (stored next to `pmv_editor.py`) and immediately uses them.  No manual
   editing of any file is required.
5. All future sessions load the key from `pmv_key.json` automatically.

### The key file (`pmv_key.json`)

The key and IV are stored in a small JSON file in the same directory as the
script:

```json
{
  "key": "32392013ded44052ae296db75bf03377",
  "iv":  "774abcf022b64aa193d519726f0144bd"
}
```

- If `pmv_key.json` is present, it takes priority over the built-in defaults.
- If it is missing (e.g. on a fresh install), the built-in defaults are used.
- The file is listed in `.gitignore` and should not be committed to version
  control, since each user should generate their own by running `--extract-key`.

### How key extraction works

The downloader is a .NET (C#) application.  The AES key and IV are stored as
static `readonly byte[]` fields whose initial values are embedded directly in
the PE file in a section called the **FieldRVA table**.

At runtime, the class constructor initialises these fields using a standard
.NET pattern:

```
ldtoken  <FieldRVA field>          ← IL opcode 0xD0: push a reference to the raw data
call     RuntimeHelpers.InitializeArray  ← copy raw bytes into the managed array
```

The extractor parses the .NET metadata, finds the constructor, scans its IL
bytecode for `ldtoken` instructions that reference FieldRVA-backed fields, and
reads 16 bytes from the PE file at each referenced address.  The first
reference is the Key; the second is the IV.

---

## How the project was built

The file format and encryption key were reverse-engineered from
`InvestorDatabaseDownloader.exe`, which is freely downloadable from the
[Sigma Metalytics website](https://www.sigmametalytics.com/pages/database-updaters).

The process:
1. Identified the `.dat` files as AES-encrypted by their near-maximum entropy.
2. Extracted the Windows `.exe` from its `.msi` installer using the `olefile`
   and `libarchive` Python libraries.
3. Found references to `RijndaelManaged` (the .NET AES class) in the
   executable's strings.
4. Used `dnfile` to parse the .NET assembly metadata and IL bytecode, locating
   the actual 16-byte key and IV in the FieldRVA table.
5. Confirmed decryption by recovering the readable plaintext header.
6. Determined the binary record layout by cross-referencing decrypted values
   against the known specific gravities of gold (19.3 g/cm³) and silver
   (10.49 g/cm³).
