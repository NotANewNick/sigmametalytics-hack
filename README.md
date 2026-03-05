# PMV Database Editor

A Python tool for reading, editing, and creating the encrypted `.dat` database
files used by **Sigma Metalytics Precious Metal Verifier (PMV)** devices.

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

# Create a new blank database
python pmv_editor.py --new MyNewDatabase.dat

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
  a              Add a brand-new record to the database
  d [#]          Delete a record permanently
  c [#]          Duplicate a record as a starting point for a new one
  s              Save all changes back to the current file
  w              Save all changes to a different file (leaves original intact)
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

### Add (`a`)

Prompts you for all fields of a new record, then asks where in the list to
insert it (default: at the end).

### Delete (`d [#]`)

Shows the full record detail first so you can confirm what will be removed,
then asks for confirmation before deleting.  This cannot be undone without
reloading from the original file.  You can include the record number directly
(e.g. `d 3`).

### Copy (`c [#]`)

Duplicates an existing record and inserts it immediately after the original,
named `<original name> (copy)`.  Use `e` afterwards to rename and adjust the
copy.  You can include the record number directly (e.g. `c 7`).

### Save (`s`) and Save as (`w`)

`s` overwrites the file that was originally opened.  `w` prompts for a new
filename, leaving the original file untouched.

---

## Command-line reference

```
python pmv_editor.py [options] [file.dat]
```

| Option | Description |
|---|---|
| *(no arguments)* | Prompts you to enter a `.dat` file path |
| `file.dat` | Opens the specified `.dat` file |
| `--new [file.dat]` | Creates a blank database (default name: `new_database.dat`) |
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
| `ResGreenLeft` | Resistance threshold, green LED, left probe |
| `ResYellowLeft` | Resistance threshold, yellow LED, left probe |
| `ResGreenRight` | Resistance threshold, green LED, right probe |
| `ResYellowRight` | Resistance threshold, yellow LED, right probe |
| `Field5` | Device-internal value (exact purpose not fully documented) |
| `SpecificGravity` | Specific gravity of the metal in g/cm³ |
| `DimensionModePlusTolerance` | Upper size tolerance in dimension mode |
| `DimensionModeMinusTolerance` | Lower size tolerance in dimension mode |
| `TotalWeightMultiplier` | Multiplier applied to the total weight reading |

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

Sigma Metalytics may update the key in a future release of their downloader
software.  To extract the new key automatically:

1. Download the new installer from the Sigma Metalytics website.
2. Extract `InvestorDatabaseDownloader.exe` from the installer.
3. Run:
   ```bash
   python pmv_editor.py --extract-key InvestorDatabaseDownloader.exe
   ```
4. If the key has changed, the tool will display the new values and offer to
   apply them for the current session.
5. To make the change permanent, update the `KEY` and `IV` constants near the
   top of `pmv_editor.py`.

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
