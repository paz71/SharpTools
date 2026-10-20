# SharpTools

**SharpTools** is a desktop program with tools for the BASIC programs of Sharp pocket computers (PC-1245 … PC-1600, PC-E500, PC-G850 and more). One window, one tab per tool:

| Tab | What it does |
|---|---|
| **Tokenize / Detokenize** | Converts tokenized BASIC images (`.bin`) to readable text and back |
| **Renumber** | Renumbers a BASIC listing and updates all jumps |
| **Line ends / EOF** | Adds the end-of-file mark SUB (1A) and switches line ends between LF and CR LF |
| **Tape files** | Takes the header and checksums off a `.tap` file (and checks them), or wraps an image into a tape file |
| **Program info** | Number of lines, size in bytes, variables used, which line calls which, dead jumps |

More tools can be added as plugins (see below). Python 3, no extra packages.

## Start

```
python3 sharptools.py
```

`sharptools.py` is the program you work with. It needs `tkinter` (included with most Python installs) and runs the other scripts for you, so keep all files in the same folder:

| File | Role |
|---|---|
| **`sharptools.py`** | **The GUI: start this one** |
| `sharp_tok.py` | Engine of the *Tokenize / Detokenize* tab |
| `sharp_renum.py` | Engine of the *Renumber* tab |
| `sharp_eol.py` | Engine of the *Line ends / EOF* tab |
| `sharp_info.py` | Engine of the *Program info* tab |
| `sharp_tap.py` | Engine of the *Tape files* tab |

The `sharp_*.py` scripts also work on their own from the command line (see the end of this page), but you do not need that to use SharpTools.

## Using SharpTools

**Tokenize / Detokenize.** Choose *Detokenize* (binary → BASIC text) or *Tokenize* (BASIC text → binary), pick the input file and press **Run**. For detokenizing the pocket computer is detected automatically; for tokenizing choose the model (`PC-1211`, `PC-1251`, `PC-1360`, `PC-1403`, `PC-1500`, `PC-1600`, `PC-E500`, `PC-G850` …). The output name is filled in from the input name (same folder, other extension: `banana.bas` gives `banana.bin` and back). Change it if you like, or clear it to show the listing in the log instead.

**Renumber.** Choose the BASIC listing, the first line number and the step, then press **Renumber**. The output is `NAME_renum.bas` unless you change it. All jumps (`GOTO`, `GOSUB`, `THEN`, `ELSE`, `ON x GOTO/GOSUB 10,20,30`, `ON ERROR GOTO`, `RESTORE`, `RUN`, `RESUME`, `LIST`, …) follow the new numbers. Fixed names such as `GOTO "INTRO"`, the label in front of a line (`10 "A" PRINT`), strings, `REM` / `'` comments, `DATA` lines, computed jumps and hex numbers are not touched. A jump to a line that does not exist is left as it is and reported.

**Line ends / EOF.** Choose a text file. **Add SUB (1A) at end of file**, **Toggle LF ⇄ CR LF**, **All → LF** and **All → CR LF** change the file in memory and show the result in the log; nothing is written until you press **Save**, which asks for confirmation and then overwrites the file. **Reload** discards the changes. A file with CR LF (also a mixed one) can be switched to LF with *Toggle* or *All → LF*.

**Tape files.** A `.tap` file is a tape recording: a 10-byte header, the program with a checksum byte after every block of 120 bytes, and an end mark. Choose *Strip*, pick the tape file and press **Run** to get the bare image (`NAME.bin`, or `NAME_img.bin` if the tape file is itself called `.bin`) that the *Tokenize / Detokenize* tab reads. The checksums are verified and wrong blocks are reported. *Check only* shows the header, the number of blocks and the result of the check without writing anything. *Wrap* does the reverse for a bare image and writes `NAME.tap`. Reading and checking works for the PC-1260 … PC-1475 and PC-1280/1360/1470/1475 tape layout (ID 70 … 73, verified on real files) and the PC-125x layout (ID 20, 21, 80); wrapping is only available for the first one, with the header `70 00 00 00 00 00 00 00 5F 14` taken from real files (on the command line `--header-from other.tap` copies the header of another tape file).

**Program info.** Choose a BASIC listing (`.bas`) or a tokenized image (`.bin`) and press **Analyze**. The log shows the number of lines, the size of the tokenized image in bytes (what the program needs in the pocket's memory, plus a little overhead), the variables used (numbers, strings, arrays), **dead jumps** (a `GOTO`/`GOSUB`/`THEN` to a line that does not exist, or to a label that is not defined), **lines nothing jumps to** (the first line and lines with a label such as `10 "A"` are entry points and left out; lines that follow a `GOTO`/`END`/`STOP`/`RETURN` and are never jumped to are marked as probably never run), and two lists: which line calls which, and each jump target with its callers. *Brief* leaves the long lists out. The model decides the image size; for a listing it is `PC-1403` unless you choose another, for a `.bin` image it is detected. Text in strings, `REM` / `'` comments and `DATA` lines are not analysed, computed jumps (`GOTO A*10`) are listed but not checked.

**Default folders.** The *Settings* menu sets a default input folder (where the Browse dialogs for input files start) and a default output folder (where output names are suggested and the Browse dialogs for output files start). Without a default output folder the output Browse dialogs start in the default input folder, and a suggested output name stays next to its input file. *Show default folders* and *Clear default folders* are in the same menu. The folders are remembered in `.sharptools.json` in your home folder.

**Log.** The lower part of the window shows what each tool did, including warnings (for example about damaged lines or missing jump targets).

## Adding your own tool (plugin)

`sharptools.py` loads every file named `sharptools_<name>.py` that sits next to it and shows it as a new tab. The file defines a class `Plugin` with a `title` and a `build(parent)` method; the shared services (input/output file rows with Browse buttons, default folders, running a script in the background, the log) are available as `self.ctx`. The header of `sharptools.py` has a short example.

## Supported pocket computers

| Family | Models | Status |
|---|---|---|
| PC-1403 | 1403, 1425, 1440, 1445, 1460; 1100, 1260–1262, 1350, 1401, 1402, 1421, 1430, 1431, 1450 | 1403 verified on a real file, the others from Pocket Tools tables |
| PC-1360 | 1280, 1360, 1470, 1475 | Key tokens checked with real samples |
| PC-125x | 1245–1255, 1140, 1150, 1210–1212 | Verified on 19 real PC-1251 programs |
| PC-1500 | 1500 | Verified byte for byte |
| PC-1600 | 1600 | Verified byte for byte |
| PC-E / PC-G | E500, E500S, E550, E650, U6000, 1480U/1490U, E200, E220, G801–G850V | Decoding complete, tokenizing experimental |

PC-E500S / G850VS keywords (`WHILE`, `WEND`, `SWITCH`, `CASE`, `DEFAULT`, `ENDSWITCH`, `ENDIF`, `REPEAT`, `UNTIL`, `ELSE`) are included. Special characters (katakana, card suits, Π, √ and so on) are written as Unicode and read back to the same bytes. Damaged files do not stop detokenizing: problems are reported and the rest is still decoded.

## Command line (optional)

The engines can be used without the GUI. The model name is written as `PC-1251` (the `PC-` prefix and the case are optional). The output file is optional everywhere.

```
python3 sharp_tok.py det [--model M] input.bin [output.bas]     # image -> text (model detected if omitted)
python3 sharp_tok.py tok [--model M] input.bas [output.bin]     # text -> image (default model PC-1403)
python3 sharp_renum.py input.bas [output.bas] [--start 10] [--step 10]
python3 sharp_eol.py sub    input [output] [--inplace]          # add SUB (1A) at the end
python3 sharp_eol.py toggle input [output] [--inplace]          # any CR -> LF, otherwise LF -> CR LF
python3 sharp_eol.py lf     input [output] [--inplace]          # all line ends LF
python3 sharp_eol.py crlf   input [output] [--inplace]          # all line ends CR LF
python3 sharp_eol.py info   input                               # show line ends and SUB
python3 sharp_tap.py info|strip|wrap input [output]                # tape files (.tap)
python3 sharp_info.py input.bas [--model PC-1403] [--brief]     # program info (also input.bin)
```

- Without an output file `sharp_tok.py` writes next to the input with the other extension (`banana.bas` ⇄ `banana.bin`), `sharp_renum.py` writes `NAME_renum.bas`, and `sharp_eol.py` writes `NAME_sub.ext`, `NAME_lf.ext` or `NAME_crlf.ext` (`--inplace` changes the input itself). `_out` is added if a name would equal the input, so the input is never overwritten by accident. Use `-` as output to print the result.
- `sharp_tok.py`: `--model` is the only option. Example: `10 PRINT "HI"` and `20 GOTO 10` become `00 0A 06 DE 22 48 49 22 0D 00 14 04 C6 31 30 0D FF` for a PC-1403.
- `sharp_eol.py toggle`: a file with CR LF (or any CR, also mixed line ends) becomes LF, a file with LF only becomes CR LF. `lf` and `crlf` set the line ends explicitly. Binary files (with `00` bytes) are refused. `sub` does nothing if the file already ends with SUB.

## Credits and license

Tables and character codes follow **Pocket Tools** (`bas2img`, `wav2bin`) and the Sharp manuals. The PC-1500/1600 tables derive from **SharpDataExchange** by Martin Erzberger (Polyform Noncommercial 1.0.0), so this project is for **noncommercial use only**.


