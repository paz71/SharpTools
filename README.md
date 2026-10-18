# Sharp Pocket BASIC Tools

Tokenize, detokenize and renumber BASIC programs of Sharp pocket computers (PC-1245 … PC-1600, PC-E500, PC-G850 and more). Python 3, no dependencies.

| File | Purpose |
|---|---|
| `sharp_tok.py` | Tokenizer / detokenizer (command line) |
| `sharp_renum.py` | Renumbers a BASIC listing (command line) |
| `sharp_eol.py` | Line ends (LF / CR LF) and the end-of-file mark SUB (command line) |
| `sharptools.py` | One window for all tools, one tab per tool (needs `tkinter`, which comes with most Python installs) |

Keep the files in the same folder.

## Quick start

**GUI:** run `python3 sharptools.py`. Each tool is a tab:

- *Tokenize / Detokenize*: pick the mode, choose the input file (and the model for tokenizing), press **Run**. For detokenizing the model is detected automatically. The output name is filled in from the input name (same folder, other extension); you can change it, or clear it to show the listing in the log instead.
- *Renumber*: choose the BASIC file, the first line number and the step, press **Renumber**. The output name is `NAME_renum.bas` unless you change it.
- *Line ends / EOF*: choose a text file, then press **Add SUB (1A) at end of file** and/or **Toggle LF ⇄ CR LF**. The buttons change the file in memory and show the result in the log; nothing is written until you press **Save**, which asks for confirmation and then overwrites the file. **Reload** discards the changes.

**Default folders:** the *Settings* menu sets a default input folder (where the Browse dialogs for input files start) and a default output folder (where output names are suggested and the Browse dialogs for output files start). If no default output folder is set, the Browse dialogs for output start in the default input folder, and a suggested output name stays next to its input file. *Show default folders* and *Clear default folders* are in the same menu. The folders are remembered in `.sharptools.json` in your home folder.

**Command line:**

```
python3 sharp_tok.py det [--model M] input.bin [output.bas]    # image -> text
python3 sharp_tok.py tok [--model M] input.bas [output.bin]      # text -> image
```

For example:

```
python3 sharp_tok.py tok --model PC-1251 prog.bas prog.bin
python3 sharp_tok.py det --model PC-1360 prog.bin prog.bas
```

- `det`: `--model` is optional (the model family is detected). Blank lines are left out.
- **Output file is optional** in both directions: it gets the name and folder of the input with the other extension, e.g. `/home/user/banana.bas` gives `/home/user/banana.bin`, and `banana.bin` gives `banana.bas`. An existing file with that name is overwritten. Use `-` as output (`det banana.bin -`) to print the listing on the screen instead.
- `tok`: `--model` defaults to `1403`.
- A PC-1500 image that contains PC-1600 codes (line targets stored as `1F hi lo 00`, `LINE` as `F0 99`) is decoded with the PC-1600 table, with a note on the screen. Use `--model PC-1600` for such programs when tokenizing.
- `--model` is the only option. It selects the pocket computer by its name: `PC-1211`, `PC-1251`, `PC-1360`, `PC-1403`, `PC-1500`, `PC-1600`, `PC-E500`, `PC-G850` and so on. The `PC-` prefix and the case are optional (`pc-1211`, `1211` and `PC-1211` are the same; the GUI lists the names as `PC-1211`, `PC-1251` and so on). Models of one family give the same result, so the exact machine matters only where it is noted below.

Example: the lines `10 PRINT "HI"` and `20 GOTO 10` become `00 0A 06 DE 22 48 49 22 0D 00 14 04 C6 31 30 0D FF` for a PC-1403.

## Renumbering (`sharp_renum.py`)

```
python3 sharp_renum.py input.bas [output.bas] [--start 10] [--step 10]
```

Renumbers a text listing (for example one made with `sharp_tok.py det`). The new numbers start at `--start` and grow by `--step` (default 10 and 10). Without an output file the result is written as `NAME_renum.bas` (`-` prints it).

All jumps are updated: `GOTO`, `GOSUB`, `THEN`, `ELSE`, `ON x GOTO/GOSUB 10,20,30`, `ON ERROR GOTO`, `RESTORE`, `RUN`, `RESUME`, `LIST`, `AUTOGOTO`, `ARUN`.

**Not changed:** names in quotes such as `GOTO "INTRO"` (the fixed labels of the pocket computers) and the label in front of
