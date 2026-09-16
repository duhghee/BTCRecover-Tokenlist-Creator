#  BTCRecover Tokenlist Creator GUI

## 1. Purpose

`tokenlist_creator.py` is a Python/Tkinter desktop GUI for creating `tokenlist.txt` files. It does **not** perform wallet recovery itself. Its job is to turn a 12-position configuration into tokenlist text that can be consumed by compatible recovery software.

The program supports:

- Fixed-position tokens using `^POSITION^word`
- Unanchored candidate-set lines
- Single-word candidates
- Space-separated custom candidate sets
- Candidates selected from a wordlist by word length
- All words from a supplied wordlist
- Candidate-count display
- Tokenlist preview
- JSON template save/load
- Generation of `tokenlist.txt`

The GUI contains exactly 12 configurable rows (`ROWS = 12`).

## 2. Requirements

- Python 3
- Tkinter
- A plain-text wordlist when using **Words by length** or **All wordlist words**
- Read/write access to locations used for templates and generated tokenlists

On Ubuntu/Debian, Tkinter may be installed with:

```bash
sudo apt install python3-tk
```

No third-party Python packages are imported by the program.

## 3. Program Structure

### Constants

```python
APP_TITLE = "BTCRecover Tokenlist Creator"
ROWS = 12
```

`ROWS` determines the number of token positions shown in the GUI.

### Main class

```python
class TokenlistGUI(tk.Tk):
```

The application is implemented as a `tk.Tk` subclass. Construction initializes the main window, the wordlist path, status text, the row configuration list, and then builds the interface.

### Main entry point

```python
if __name__ == "__main__":
    TokenlistGUI().mainloop()
```

Running the script directly creates the GUI and starts the Tk event loop.

## 4. GUI Data Model

Each of the 12 rows stores five Tkinter variables:

| Field | Purpose |
|---|---|
| `behavior` | `Fixed` or `Permute` |
| `source` | Determines where candidates come from |
| `value` | Single word or space-separated custom words |
| `length` | Selected length for wordlist filtering |
| `count` | Display-only candidate count |

The available source modes are:

- `Single word`
- `Custom words`
- `Words by length`
- `All wordlist words`

## 5. Fixed vs. Permute Behavior

### Fixed

A Fixed row anchors every candidate to that row's position.

For position 3:

```text
tiger
```

becomes:

```text
^3^tiger
```

Multiple alternatives:

```text
tiger lion bear
```

become:

```text
^3^tiger ^3^lion ^3^bear
```

This behavior is implemented in `build_text()`.

### Permute

A Permute row is emitted without a position anchor.

For example:

```text
use used usage useful useless
```

is written exactly as an unanchored candidate-set line.

The GUI's own help text states that XRPrecover is expected to permute these unanchored candidate-set lines only among positions that have no fixed candidates. The creator itself does not perform this permutation; it only writes the tokenlist.

## 6. Candidate Sources

### Single word

`tokens_for_row()` returns one candidate when the Value field is non-empty.

Example:

```text
zoo
```

### Custom words

The Value field is split on whitespace.

Example:

```text
use used usage useful useless
```

produces five candidates.

### Words by length

The configured wordlist is loaded and filtered with:

```python
[w for w in wordlist if len(w) == n]
```

This is literal Python string length. No BIP39-specific validation is performed by this function.

### All wordlist words

Every non-empty line loaded from the selected wordlist becomes a candidate.

## 7. Wordlist Handling

The default wordlist path is:

```text
english.txt
```

`read_wordlist()`:

1. Expands `~` in the path.
2. Checks whether the path is a file.
3. Opens it as UTF-8 with replacement for decoding errors.
4. Strips each line.
5. Ignores empty lines.

If the path is invalid, it returns an empty list rather than raising a user-facing exception.

The GUI provides **Browse…** and **Reload** controls for selecting and recounting the wordlist.

## 8. Candidate Counts and Status

`update_counts()` calculates the number of candidates for every row.

The status bar reports:

- Number of words loaded from the wordlist
- Sum of configured row candidate counts
- Number of non-empty Fixed rows
- Number of non-empty Permute rows

Important: **configured candidates is not the total number of recovery combinations.** It is only the sum of the candidate counts configured across rows.

## 9. Tokenlist Generation Algorithm

`build_text()` performs the core conversion.

For positions 1 through 12:

1. Resolve the row's candidates with `tokens_for_row()`.
2. Skip the row if it has no candidates.
3. If Fixed, prefix every candidate with `^position^`.
4. If Permute, leave candidates unanchored.
5. Join candidates for that row with spaces.
6. Join configured rows with newline characters.
7. Add a final newline when output is non-empty.

Example:

```text
^1^word
^2^word
^3^word
word
word word word word 
word word 
```

## 10. Preview

The **Preview** button calls `build_text()` and displays the generated text in a separate read-only Tkinter Text window.

This is useful for checking anchors and candidate-set lines before writing the file.

Preview does not write anything to disk.

## 11. Generating tokenlist.txt

The **Generate tokenlist.txt** button:

1. Builds the current tokenlist.
2. Refuses to continue if there are no configured tokens.
3. Opens a Save As dialog.
4. Defaults to `tokenlist.txt`.
5. Writes UTF-8 text.
6. Updates the status bar.
7. Displays a completion message.

## 12. Template Format

The GUI can save its configuration as JSON.

Default filename:

```text
tokenlist_template.json
```

The format is structurally equivalent to:

```json
{
  "wordlist": "english.txt",
  "rows": [
    {
      "behavior": "Fixed",
      "source": "Single word",
      "value": "special",
      "length": "5"
    }
  ]
}
```

Each row records:

- behavior
- source
- value
- length

Candidate counts are not stored because they are recalculated.

## 13. Loading Templates

**Load Template…** reads a JSON file and restores the saved settings.

The program uses defaults for missing properties:

- behavior → `Fixed`
- source → `Single word`
- value → empty string
- length → `5`

Rows are restored with `zip(self.rows, data.get("rows", []))`, so at most the GUI's 12 rows are populated.

After loading, candidate counts are recalculated.

## 14. Clear All

**Clear All** resets all 12 rows to:

```text
Behavior: Fixed
Source: Single word
Value: [empty]
Length: 5
```

It then recalculates the displayed counts.

## 15. Error Handling

The program contains basic GUI error handling:

- Missing wordlist → empty wordlist
- Invalid length conversion → no candidates for that row
- Empty generated tokenlist → warning
- Template-load failure → error dialog
- Count/update exception → error text in status bar

It does not validate whether candidate words are valid mnemonic words, whether a generated recovery candidate has a valid checksum, or whether it derives a desired wallet address.

## 16. Security and Scope

This program only creates text configuration files. It does not:

- derive wallet addresses
- validate mnemonic checksums
- test recovery combinations
- store private keys
- connect to a network
- perform wallet recovery

Nevertheless, tokenlists can contain sensitive recovery information. Treat generated tokenlists and saved templates as sensitive files when they contain real wallet-recovery words.

## 17. Compatibility Notes

The output format produced by the application consists of anchored and unanchored token lines. Actual interpretation is determined by the recovery program that consumes the generated file.

Before a large recovery run, use **Preview** and perform a small known test to verify that the target recovery program interprets:

- `^POSITION^word` anchors as expected
- unanchored candidate-set lines as expected
- the desired relationship between Fixed and Permute positions

## 18. Function Reference

| Method | Responsibility |
|---|---|
| `_build_ui()` | Creates the complete interface and startup examples |
| `browse_wordlist()` | Selects a text wordlist |
| `read_wordlist()` | Loads non-empty wordlist lines |
| `tokens_for_row()` | Resolves candidates for one row |
| `update_counts()` | Updates row counts and status |
| `build_text()` | Converts GUI state into tokenlist text |
| `preview()` | Displays generated tokenlist without saving |
| `generate()` | Saves `tokenlist.txt` |
| `clear_all()` | Resets all rows |
| `save_template()` | Saves GUI configuration to JSON |
| `load_template()` | Restores GUI configuration from JSON |

## 19. Data Flow

```text
Wordlist file ─────┐
                   ├─> tokens_for_row()
GUI row settings ──┘
                         |
                         v
                    build_text()
                    /         \\
                   v           v
               Preview     tokenlist.txt

GUI row settings ──> Save Template ──> JSON
JSON ──────────────> Load Template ──> GUI row settings
```

## 20. Reference File

'''english_by_length.txt''' Has no function. Provided for reference.

## 21. Recommended Workflow

1. Start `tokenlist_creator.py`.
2. Select the correct wordlist.
3. Click **Clear All** if you do not want the examples.
4. Enter every known fixed position.
5. Add alternatives with **Custom words** where necessary.
6. Configure intentionally unanchored rows as **Permute**.
7. Use **Words by length** or **All wordlist words** where appropriate.
8. Check the candidate counts.
9. Click **Preview**.
10. Verify the anchors and unanchored lines carefully.
11. Save a JSON template if you want to reuse the setup.
12. Generate `tokenlist.txt`.
13. Test the resulting tokenlist with a known test case before starting a large recovery job.

## 22. Troubleshooting

### Candidate count is 0 for Words by length

Check that:

- the Wordlist path is correct
- the file exists
- the selected length actually occurs in the wordlist

Click **Browse…** and reselect the file if necessary.

### All wordlist-driven rows show 0

The wordlist probably was not loaded. Look at the status bar. It reports the number of loaded words.

### A word is appearing at a fixed position when I wanted it unanchored

Change that row's Behavior from **Fixed** to **Permute**.

### A word is unanchored when I know its exact position

Change the row to **Fixed**.

### Preview is empty

No row currently resolves to any candidates. Add values or load a usable wordlist.

### Template will not load

The program expects JSON in the format produced by **Save Template…**. A malformed or incompatible JSON file produces an error dialog.

## 23. Sensitive Data Warning

A real tokenlist or saved template may reveal words associated with wallet recovery. Store these files securely and avoid sharing them unnecessarily.

The GUI itself does not perform recovery or connect to the network.
