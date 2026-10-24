# SharpTools

**SharpTools** is a desktop program with tools for the BASIC programs of Sharp pocket computers (PC-1245 … PC-1600, PC-E500, PC-G850 and more). One window, one tab per tool:

| Tab | What it does |
|---|---|
| **Tokenize / Detokenize** | Converts tokenized BASIC images (`.bin`) to readable text and back |
| **Renumber** | Renumbers a BASIC listing (or only a range of lines) and updates all jumps; merges two programs |
| **Line ends / EOF** | Adds the end-of-file mark SUB (1A) and switches line ends between LF and CR LF |
| **Tape files** | Takes the header and checksums off a `.tap` file (and checks them), or wraps an image into a tape file |
| **Compare** | Shows the differences between two listings, two images or a listing and an image |
| **Convert model** | Converts the display POKEs and the CALL that activates the display of a program from one pocket computer to another (PC-1401/02, PC-140x/1460, PC-1475, PC-125x, PC-126x); lists source and target statements |
| **Minify / Tidy** | Removes comments and unneeded spaces to save memory, or formats a listing so it is easier to read |
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
| `sharp_comp.py` | Engine of the *Compare* tab |
| `sharp_rem.py` | Engine of the *Minify / Tidy* tab |
| `sharp_convert.py` | Engine of the *Convert model* tab |
| `icon.png` | Window icon of `sharptools.py` (optional, must be in the same folder) |

The `sharp_*.py` scripts also work on their own from the command line (see the end of this page), but you do not need that to use SharpTools.

## Using SharpTools

**Tokenize / Detokenize.** Choose *Detokenize* (binary → BASIC text) or *Tokenize* (BASIC text → binary), pick the input file and press **Run**. For detokenizing the pocket computer is detected automatically; for tokenizing choose the model (`PC-1211`, `PC-1251`, `PC-1360`, `PC-1403`, `PC-1500`, `PC-1600`, `PC-E500`, `PC-G850` …). The output name is filled in from the input name (same folder, other extension: `banana.bas` gives `banana.bin` and back). Change it if you like, or clear it to show the listing in the log instead.

**Renumber.** Choose the BASIC listing, the first line number and the step, then press **Renumber**. The output is `NAME_renum.bas` unless you change it. All jumps (`GOTO`, `GOSUB`, `THEN`, `ELSE`, `ON x GOTO/GOSUB 10,20,30`, `ON ERROR GOTO`, `RESTORE`, `RUN`, `RESUME`, `LIST`, …) follow the new numbers. Fixed names such as `GOTO "INTRO"`, the label in front of a line (`10 "A" PRINT`), strings, `REM` / `'` comments, `DATA` lines, computed jumps and hex numbers are not touched. A jump to a line that does not exist is left as it is and reported.

*Only a range of lines:* fill in **From line** and/or **To line** and only those lines get new numbers; all other lines keep theirs, and jumps to the renumbered lines follow, also from lines outside the range. The new numbers must fit between the line before and the line after the range, otherwise you get a message with the free numbers. If *First line number* is empty the range keeps its first line number.

*Merge two programs:* choose the second program under **Merge another program behind this one**. Both programs are renumbered in one go: the first from *First line number* (default 10) by *Step*, the second continues directly below it, also by *Step*; all jumps follow. A jump in the second program to a line of the first program is reported. With **Keep both numberings** nothing is renumbered (a normal merge keeping the line numbers of both listings), but no line number may exist in both programs. The result is `NAME_merged.bas`. A merge cannot be combined with a range.

**Line ends / EOF.** Choose a text file. **Add SUB (1A) at end of file**, **Toggle LF ⇄ CR LF**, **All → LF** and **All → CR LF** change the file in memory and show the result in the log; nothing is written until you press **Save**, which asks for confirmation and then overwrites the file. **Reload** discards the changes. A file with CR LF (also a mixed one) can be switched to LF with *Toggle* or *All → LF*.

**Tape files.** A `.tap` file is a tape recording: a 10-byte header, the program with a checksum byte after every block of 120 bytes, and an end mark. Choose *Strip*, pick the tape file and press **Run** to get the bare image (`NAME.bin`, or `NAME_img.bin` if the tape file is itself called `.bin`) that the *Tokenize / Detokenize* tab reads. The checksums are verified and wrong blocks are reported. *Check only* shows the header, the number of blocks and the result of the check without writing anything. *Wrap* does the reverse for a bare image and writes `NAME.tap`. The tape layout is taken from the ID byte of the file: PC-1260 … PC-1475 and PC-1280/1360/1470/1475 (ID 70 … 73, verified on real files) and the PC-125x layout (ID 20, 21, 80). *Wrap* has a **Model** menu and a **Tape name** (7 characters at most): the model decides the ID byte and layout (PC-125x family ID 20, PC-1211 ID 80, PC-126x, PC-1350/1450, PC-1401/02 and PC-140x/1460 ID 70, PC-1280/1360/1475 ID 72), as written by `bin2wav.c` of Pocket Tools. Only the ID 70 layout was checked against real files; the others follow the `bin2wav.c` source and should be tested on your emulator or machine. Not supported: PC-1500, PC-1600, PC-E/G series and binary or data files. On the command line `--model 1251`, `--name NAME` and `--header-from other.tap` (copy the header of another tape file) are available.

**Compare.** Choose program A and program B (a BASIC listing, a tokenized image or a tape file each; images are detected and decoded) and press **Compare**. Lines are matched by their line number: the log lists lines that are only in A, lines that are only in B and lines with the same number but different text. Tick *Ignore line numbers (renumbered program)* to compare a program with a renumbered copy of itself, or after lines were inserted or deleted: both programs are renumbered 1, 2, 3 … internally (jump targets follow) and compared line by line, and the original line numbers are shown. *Ignore spaces and case* helps when a hand-typed listing is compared with a detokenized one. Two images are also compared byte by byte (size, differing bytes with their offsets) before the listings are compared. For images the model is detected unless you choose one.

**Convert model.** Converts only what depends on the display of the pocket computer: the `POKE` statements into the display memory and the `CALL` that activates the display. The rest of the program is not touched and not checked (no keyword or screen check). Choose a listing (`.bas`) or an image (`.bin`, also a `.tap` tape file), the source model under **From** (a listing needs it, an image can leave it empty), the target under **To**, and press **Convert**. Only the pocket computers whose display addresses are known can be chosen: PC-1401/02, PC-140x/1460 (1403, 1403H, 1460), PC-1475, PC-125x and PC-126x. A two-line display (PC-126x, PC-1475) can only be converted to another two-line display, so choosing PC-126x as **From** selects PC-1475 as **To** and the other way round; the one-line models convert among each other.

- If the program has no `POKE` into the display memory and no activation `CALL`, only a message is shown and no file is written.
- A `POKE` into the display memory is moved to the address of the same line, display position and column on the target. The models have different address blocks (PC-1403 six, PC-1475 four, PC-125x and PC-126x two, PC-1401/02 two): where the bytes of one `POKE` are not continuous on the target, the `POKE` is split into several `POKE` statements.
- A `POKE` whose address is a variable or an expression (`POKE Z,X`, `POKE &2040+I,X`) is followed through the program. A variable that is only used as POKE address gets its assignments changed (`Z=&2878` becomes `Z=&2A78`); a variable that is set once and used elsewhere is replaced by the new address in the `POKE`; `&2040+I` gets a new first term (`&2A00+I`) if the display area is a plain shift on the target. If the address cannot be determined (the variable changes in loops or with `INPUT`, is set in several places and used elsewhere, …) the `POKE` is listed with the reason.
- The `CALL` that activates the display (`&05A2` on the PC-1401/02, `&04B8` on the PC-1403, `&11E0` on the PC-125x) is changed to the call of the target. The PC-1475 and PC-126x need none: it is removed (a line that held only the call is deleted, or stays as `REM` if a jump goes to it). If the target needs a call and the program has none, a line with the call is added in front of the first display `POKE` (in the same line if no line number is free before it or it is jumped to).
- The log shows, for every changed statement, the source and the target statement, and lists what could not be changed (a `POKE` inside `IF … THEN`, bytes the target has no place for, `POKE` / `CALL` with an address in a variable). **The result has to be reviewed manually**; the log says so. Different graphics positions (PC-125x position 1, PC-1403 position 24) are reported.
- The output has the type of the input: a listing gives a listing (`NAME_pc125x.bas`), an image gives an image (`NAME_pc125x.bin`; it is read with the tokens of the source and written with the tokens of the target). On the command line: `sharp_convert.py input --to MODEL [output] [--from MODEL] [--listing FILE]`.

**Model menus.** The model menus of the other tabs show families: PC-121x, PC-125x, PC-126x, PC-1401/02, PC-140x/1460, PC-144x, PC-145x, PC-1350, PC-1360, PC-1475, PC-150x, PC-1600, PC-E5xx, PC-G8xx and PC-E2xx. A family stands for models with the same token table; PC-E5xx works like the PC-E500S. On the command line the single names (PC-1251, PC-1403, PC-E500 …) and the family names (quote names with a `/`) both work. *Convert model* lists only the five models with known display addresses.

**Minify / Tidy.** Choose *Minify* to save memory on the pocket computer. The options remove lines that hold only a comment (`REM …` or `' …`), comments at the end of a line (`:REM …`), and the spaces that are not needed (`PRINT A`, `FOR I=1 TO N` and the space between two words stay; spaces around `=`, `+`, `:` and commas go). A jump to a removed line is changed to the next line that stays; a comment line with a label (`10 "A" REM …`) is never removed, and neither is a comment right after `THEN` / `ELSE`. The log shows the size of the tokenized image before and after, for the chosen model (default `PC-1403`). *Tidy* makes a listing easier to read: a space after and before keywords (`FORI=1TO10` becomes `FOR I=1 TO 10`), spaces around `=` `<` `>`, one space after `:` `,` `;`. Strings, comments and `DATA` lines are not touched, and only spaces and comments ever change, never the program text. The output is `NAME_min.bas` or `NAME_tidy.bas`.

**Program info.** Choose a BASIC listing (`.bas`) or a tokenized image (`.bin`) and press **Analyze**. The log shows the number of lines, the size of the tokenized image in bytes (what the program needs in the pocket's memory, plus a little overhead), the variables used (numbers, strings, arrays), **dead jumps** (a `GOTO`/`GOSUB`/`THEN` to a line that does not exist, or to a label that is not defined), **lines nothing jumps to** (the first line and lines with a label such as `10 "A"` are entry points and left out; lines that follow a `GOTO`/`END`/`STOP`/`RETURN` and are never jumped to are marked as probably never run), and two lists: which line calls which, and each jump target with its callers. *Brief* leaves the long lists out. The model decides the image size; for a listing it is `PC-1403` unless you choose another, for a `.bin` image it is detected. Text in strings, `REM` / `'` comments and `DATA` lines are not analysed, computed jumps (`GOTO A*10`) are listed but not checked.

**Default folders.** The *Settings* menu sets a default input folder (where the Browse dialogs for input files start) and a default output folder (where output names are suggested and the Browse dialogs for output files start). Without a default output folder the output Browse dialogs start in the default input folder, and a suggested output name stays next to its input file. *Show default folders* and *Clear default folders* are in the same menu. The folders are remembered in `.sharptools.json` in your home folder.

**Log / Result.** Every tab has its own *Log / Result* pane below its controls. It shows what the tool of that tab did, including warnings (for example about damaged lines or missing jump targets), and it keeps its text when you change tabs. A result always appears in the tab that started the run, and *Clear log* clears only that tab.

**Files in all tabs.** Every input field has a drop-down list. A file you choose or type in one tab is offered in the input fields of all other tabs, newest first. The files that a tool writes (Detokenize, Renumber, Tape files, Minify / Tidy, Convert model, and *Save* in Line ends / EOF) are added to the same list after a successful run, so you can use them straight away as input for the next step, for example Strip a tape file, Detokenize the image, Minify the listing, then Convert it. A file written by *Tokenize* (a tokenized image) is not offered in the other tabs. The log says *Available as input file in all tabs* when this happened. The list is kept while the program runs.


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
python3 sharp_renum.py input.bas [output.bas] [--start 10] [--step 10] [--from N] [--to M]
python3 sharp_renum.py input.bas [output.bas] --merge other.bas [--start N] [--step N] [--keep-numbers]
python3 sharp_eol.py sub    input [output] [--inplace]          # SUB (1A) only as the very last byte (removes other SUBs)
python3 sharp_eol.py toggle input [output] [--inplace]          # any CR -> LF, otherwise LF -> CR LF
python3 sharp_eol.py lf     input [output] [--inplace]          # all line ends LF
python3 sharp_eol.py crlf   input [output] [--inplace]          # all line ends CR LF
python3 sharp_eol.py info   input                               # show line ends and SUB
python3 sharp_tap.py info|strip|wrap input [output] [--model N --name NAME]                # tape files (.tap)
python3 sharp_comp.py A B [--renumbered] [--loose] [--model M]   # compare two programs
python3 sharp_convert.py input --to PC-1360 [output.bin] [--from M] [--force] [--strict] [--listing F.bas]   # convert to another model
python3 sharp_rem.py minify|tidy input.bas [output.bas] [--keep-rem-lines] [--keep-comments] [--keep-spaces] [--model M]
python3 sharp_info.py input.bas [--model PC-1403] [--brief]     # program info (also input.bin)
```

- Without an output file `sharp_tok.py` writes next to the input with the other extension (`banana.bas` ⇄ `banana.bin`), `sharp_renum.py` writes `NAME_renum.bas` (`NAME_merged.bas` for `--merge`), and `sharp_eol.py` writes `NAME_sub.ext`, `NAME_lf.ext` or `NAME_crlf.ext` (`--inplace` changes the input itself). `_out` is added if a name would equal the input, so the input is never overwritten by accident. Use `-` as output to print the result.
- `sharp_tok.py`: `--model` is the only option. Example: `10 PRINT "HI"` and `20 GOTO 10` become `00 0A 06 DE 22 48 49 22 0D 00 14 04 C6 31 30 0D FF` for a PC-1403.
- `sharp_eol.py toggle`: a file with CR LF (or any CR, also mixed line ends) becomes LF, a file with LF only becomes CR LF. `lf` and `crlf` set the line ends explicitly. Binary files (with `00` bytes) are refused. `sub` does nothing if the file already ends with SUB.

## Credits and license

Tables and character codes follow **Pocket Tools** (`bas2img`, `wav2bin`) and the Sharp manuals. The PC-1500/1600 tables derive from **SharpDataExchange** by Martin Erzberger (Polyform Noncommercial 1.0.0), so this project is for **noncommercial use only**.

## Changes in v0.6.3 (since v0.6.2)

- *Convert model*: the PC-126x display is now described as four blocks of 60 bytes (`&2000-&203B`, `&2040-&207B`, `&2800-&283B`, `&2840-&287B`) like the PC-1475, and the PC-1475 ranges were corrected (`&2800-&283B`, `&2A00-&2A3B`, `&2840-&287B`, `&2A40-&2A7B`). PC-126x to PC-1475 (and back) now really changes the addresses. This PC-126x layout is derived from your program and still has to be verified on real hardware.
- *Convert model*: a POKE with a label before it (`"T" POKE ...`) is recognized.
- *Convert model*: POKEs that cover several target blocks are split, and POKEs that were split on the source are joined again where the target memory is continuous.
- *Convert model*: addresses given as a variable or an expression (`POKE Z,X`) are now resolved: the value is determined from the program, replaced by the target address (assignment changed, or the base address of a base+offset shifted). What cannot be determined is listed with the reason instead of being skipped silently. Program lines are added or removed only where the target requires it (the display CALL).
- *Convert model*: bytes that cannot be mapped to the target display are reported as warnings in the list and no longer stop the conversion.
- *Renumber*, merging (`sharp_renum.py`): without *Keep both numberings* both programs are renumbered in one run. The first program is renumbered from *First line number* (10 if empty) by *Step*, the second is added directly below it, also by *Step*, and all jumps follow. With *Keep both numberings* a normal merge keeps the line numbers of both listings (an error names numbers that exist in both). The old "next full hundred" start is gone.
- *Tape files* (`sharp_tap.py`), after studying `bin2wav.c` of Pocket Tools: *Wrap* has a **Model** menu (PC-125x family, PC-1211, PC-126x, PC-1350/1450, PC-1401/02 and so on, PC-140x/1460, PC-1280/1360/1475) that sets the ID byte and the tape layout, and a **Tape name** field (7 characters at most). New layout for the old series (ID 20, 80: checksum after every 8 bytes, end mark F0). Command line: `--model`, `--name`, `--id`. Only the ID 70 layout was checked against real files; the others follow the `bin2wav.c` source and should be tested.
- *Tape files*: *Check only* and *Strip* show the ID byte with its meaning and the tape name from the header.
