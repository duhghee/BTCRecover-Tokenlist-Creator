# Bip39-Tokenlist-Creator

Python/Tkinter desktop GUI for creating tokenlist.txt files for BTCRecover-compatible recovery workflows.  It does **not** perform wallet recovery itself. Its job is to turn a 12-position configuration into tokenlist text that can be consumed by compatible recovery software.

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

## Requirements

- Python 3
- Tkinter
- A plain-text wordlist when using **Words by length** or **All wordlist words**
- Read/write access to locations used for templates and generated tokenlists

On Ubuntu/Debian, Tkinter may be installed with:

```bash
sudo apt install python3-tk
```

No third-party Python packages are imported by the program.

## Recommended Workflow

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
