# Usage Guide — BTCRecover Tokenlist Creator GUI

## What This Program Does

`tokenlist_creator.py` gives you a 12-row GUI for building a `tokenlist.txt` file.

Each row represents a position. You decide whether the row is:

- **Fixed** — its candidate word(s) stay anchored to that numbered position.
- **Permute** — its candidate-set line is written without an anchor so compatible recovery software can treat it as unanchored.

The GUI creates the tokenlist only. It does not perform wallet recovery.

## 1. Start the Program

Open a terminal in the folder containing the script:

```bash
python3 tokenlist_creator.py
```

If Tkinter is missing on Ubuntu/Debian:

```bash
sudo apt install python3-tk
```

Then run the program again.

## 2. Select a Wordlist

At the top of the GUI is the **Wordlist** field.

The default is:

```text
english.txt
```

You only need a wordlist for rows using:

- **Words by length**
- **All wordlist words**

Click **Browse…** to select another `.txt` file.

The expected format is one candidate word per line:

```text
abandon
ability
able
about
above
...
```

Click **Reload** after changing the wordlist externally.

## 3. Understand the Columns

### Pos

The numbered position, 1 through 12.

### Behavior

Choose:

**Fixed**

Use this when a candidate belongs to this exact position.

Example for position 4:

```text
word
```

generates:

```text
^4^word
```

**Permute**

Use this when the candidate line should be unanchored.

Example:

```text
word word
```

generates:

```text
word word
```

The GUI states that XRPRecover is expected to permute unanchored candidate-set lines only among positions that do not have fixed candidates.

### Source

Choose one of four candidate sources.

**Single word**

Enter one word in the Value field.

**Custom words**

Enter alternatives separated by spaces.

Example:

```text
use used usage useful useless
```

**Words by length**

Choose a length from the Length column. The GUI loads every word of exactly that character length from the selected wordlist.

**All wordlist words**

Uses every non-empty word in the selected wordlist.

### Value / custom words

Used by:

- Single word
- Custom words

It is ignored for wordlist-driven modes.

### Length

Used by **Words by length**.

For example, selecting `5` means only five-character words from the loaded wordlist are candidates for that row.

### Candidates

Shows how many candidates the current row contains.

## 4. Example: Known Fixed Words

Suppose you know the first six positions exactly.

Configure:

| Pos | Behavior | Source | Value |
|---:|---|---|---|
| 1 | Fixed | Single word | lion |
| 2 | Fixed | Single word | bear |
| 3 | Fixed | Single word | tiger |
| 4 | Fixed | Single word | zebra |
| 5 | Fixed | Single word | eagle |
| 6 | Fixed | Single word | zoo |

The preview contains:

```text
^1^lion
^2^bear
^3^tiger
^4^point
^5^zebra
^6^zoo
```

## 5. Example: Several Possible Words at One Fixed Position

Suppose position 8 is definitely position 8, but you are unsure which of five words is correct.

Set:

```text
Behavior: Fixed
Source: Custom words
Value: use used usage useful useless
```

For position 8 the output is:

```text
^8^use ^8^used ^8^usage ^8^useful ^8^useless
```

All candidates remain anchored to position 8.

## 6. Example: Unanchored / Permuting Candidates

Suppose you want this candidate set to be unanchored:

```text
garage garbage
```

Set:

```text
Behavior: Permute
Source: Custom words
Value: garage garbage
```

The generated line is:

```text
garage garbage
```

There is no `^position^` prefix.

Use **Permute** only for positions you intentionally want represented as unanchored candidate-set lines.

## 7. Example: Unknown Word of a Known Length

Suppose an unanchored candidate is known to be five characters long.

Set:

```text
Behavior: Permute
Source: Words by length
Length: 5
```

The GUI reads the selected wordlist and places all five-character words on that output line.

The **Candidates** column tells you how many words matched.

## 8. Example: Any Word in the Wordlist

Set:

```text
Source: All wordlist words
```

The entire selected wordlist becomes the row's candidate set.

This can create a very large search space when used in multiple rows.

## 9. Preview Before Saving

Click:

**Preview**

A new window shows exactly what the GUI currently intends to write.

Check that:

- Fixed rows have `^position^` prefixes.
- Permute rows do not have prefixes.
- Candidate alternatives appear on the correct lines.
- Wordlist-driven rows contain the expected candidates.

Close the Preview window when finished.

## 10. Generate tokenlist.txt

Click:

**Generate tokenlist.txt**

Choose where to save the file.

The default filename is:

```text
tokenlist.txt
```

If no rows contain candidates, the GUI warns you instead of creating an empty tokenlist.

## 11. Save Your GUI Setup as a Template

Click:

**Save Template…**

The default filename is:

```text
tokenlist_template.json
```

This saves:

- selected wordlist path
- Fixed/Permute selection
- source selection
- entered values
- selected lengths

This is useful when you want to reuse or modify a configuration later.

## 12. Load a Saved Template

Click:

**Load Template…**

Select the previously saved JSON file.

The GUI restores the saved row configuration and recalculates candidate counts.

## 13. Clear Everything

Click:

**Clear All**

All 12 rows reset to:

```text
Fixed
Single word
blank value
length 5
```

## 14. Important Difference: Fixed Alternatives vs. Permute

These two configurations are not equivalent.

### Fixed custom alternatives at position 8

```text
^8^use ^8^used ^8^usage
```

Every alternative is tied to position 8.

### Permute custom alternatives

```text
use used usage
```

The line is unanchored.

Choose **Fixed** when you know the position. Choose **Permute** when you intentionally want an unanchored candidate-set line.

## 15. Startup Values

The current version starts with example values already entered. They demonstrate both Fixed and Permute behavior.

They are not automatically your recovery data.

Either:

- replace each example with your own values, or
- click **Clear All** and build a configuration from scratch.

## 16. Recommended Workflow

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

## 17. Troubleshooting

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

## 18. Sensitive Data Warning

A real tokenlist or saved template may reveal words associated with wallet recovery. Store these files securely and avoid sharing them unnecessarily.

The GUI itself does not perform recovery or connect to the network.
