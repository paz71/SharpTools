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

**Not changed:** names in quotes such as `GOTO "INTRO"` (the fixed labels of the pocket computers) and the label in front of a line (`10 "A" PRINT`), text in strings, `REM` / `'` comments and `DATA` lines, computed jumps (`GOTO A*10`) and hex numbers (`&10`). A jump to a line that does not exist is left as it is and reported on the screen. The highest line number is 65279 (PC-125x and PC-1210/1211/1212: 999, you get a note).

## Invisible codes (`sharp_eol.py`)

```
python3 sharp_eol.py sub    input [output]    # add SUB (1A) at the end of the file
python3 sharp_eol.py toggle input [output]    # LF -> CR LF, CR LF -> LF
python3 sharp_eol.py info   input             # show line ends and whether SUB is there
```

- `sub` adds the end-of-file mark SUB (`1A`) as the last byte. A file that already ends with SUB is not changed.
- `toggle` switches the line ends: a file with CR LF gets LF, any other file gets CR LF (mixed or CR-only line ends are all replaced). Binary files (with `00` bytes) are refused.
- Without an output file a copy is written next to the input (`NAME_sub.ext`, `NAME_crlf.ext` or `NAME_lf.ext`). `--inplace` changes the input file itself.

## Adding your own tool to the GUI

`sharptools.py` loads every file named `sharptools_<name>.py` that sits next to it and shows it as a new tab. The file defines a class `Plugin` with a `title` and a `build(parent)` method; the shared services (file rows with Browse buttons, running a script in the background, the log) are available as `self.ctx`. The header of `sharptools.py` has a short example.

## Supported models

| Family | Models | Status |
|---|---|---|
| PC-1403 | 1403, 1425, 1440, 1445, 1460; 1100, 1260–1262, 1350, 1401, 1402, 1421, 1430, 1431, 1450 | 1403 verified on a real file, the others from Pocket Tools tables |
| PC-1360 | 1280, 1360, 1470, 1475 | Key tokens checked with real samples |
| PC-125x | 1245–1255, 1140, 1150, 1210–1212 | Verified on 19 real PC-1251 programs |
| PC-1500 | 1500 | Verified byte for byte |
| PC-1600 | 1600 | Verified byte for byte |
| PC-E / PC-G | E500, E500S, E550, E650, U6000, 1480U/1490U, E200, E220, G801–G850V | Decoding complete, tokenizing experimental |

PC-E500S / G850VS keywords (`WHILE`, `WEND`, `SWITCH`, `CASE`, `DEFAULT`, `ENDSWITCH`, `ENDIF`, `REPEAT`, `UNTIL`, `ELSE`) are included.

Special characters (katakana, card suits, Π, √ and so on) are written as Unicode and read back to the same bytes. Damaged files do not stop `det`: problems are reported and the rest is still decoded.

## Good to know

- **Use bare images, not `.tap` files.** A `.tap` file is a tape recording with a header and checksum bytes. Read as an image it looks damaged (wrong length warnings, odd characters in `POKE` lines). Even a file named `.bin` can be a tap file; in the PC-1403 family it starts with `70`, a bare image starts with a line number such as `00 0A`. Convert first with Pocket Tools: `wav2bin --tap file.tap --type=img`.
- PC-125x models have one code per letter, so listings are uppercase.
- If automatic detection guesses wrong on a damaged file, give `--model`. PC-1360, PC-E and PC-G share codes: use `--model E500` or `--model G850` for machine-specific keywords.
- On a PC-E500, after loading a tokenized image enter `TEXT` and then `BASIC`.

## Credits and license

Tables and character codes follow **Pocket Tools** (`bas2img`, `wav2bin`) and the Sharp manuals. The PC-1500/1600 tables derive from **SharpDataExchange** by Martin Erzberger (Polyform Noncommercial 1.0.0), so this project is for **noncommercial use only**.

## Changes in v0.3

- Output file is optional for `det` and `tok`: it gets the input name with the other extension (`banana.bas` gives `banana.bin` and back). `_out` is added if the name would equal the input. `det` no longer prints the listing without an output file; use `-` as output to print it.
- PC-1500 images with PC-1600 codes (`1F` line targets, `LINE` = `F0 99`) are detected and decoded with the PC-1600 table instead of showing `<1F>` and `<F0><99>`.
- Model names in `--help` and in the GUI use the `PC-1211`, `PC-1251`, `PC-1360` style.
- GUI: the output name now follows the input name (it was not updated after the first file), can be left empty, and is no longer required for tokenizing.

## Changes since v0.3

- New: `sharp_renum.py`, a renumbering tool (first line number and step, jump targets updated, quoted labels left alone).
- The GUI is now `sharptools.py`: one window with a tab per tool (Tokenize / Detokenize, Renumber) and a plugin interface for more tools.
- Renamed: `sharp_basic_v2.py` is now `sharp_tok.py`, `sharp_basic_gui.py` is now `sharptools.py`.
- New: `sharp_eol.py` for the invisible codes: add SUB (1A) at the end of a file and toggle the line ends between LF and CR LF. It has its own tab in `sharptools.py`, where the changes are made in memory and written with a **Save** button (after a confirmation).
- `sharptools.py`: *Settings* menu with a default input folder and a default output folder (the output folder falls back to the input folder). The *Line ends / EOF* tab now has a **Save** button that overwrites the file after a confirmation.
