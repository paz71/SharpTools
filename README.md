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
| **pc-1403 -> ML** | Compiles a PC-1403 / PC-1403H BASIC program to machine code (binary with and without header, BASIC loaders); plugin, see below |

More tools can be added as plugins (see below); the compiler *pc-1403 -> ML* is one of them. Python 3, no extra packages.

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
| `sharptools_b2ml.py` | Plugin: the *pc-1403 -> ML* tab |
| `sharp_b2ml.py` | Engine of the *pc-1403 -> ML* tab (compiler and SC61860 assembler; also works on its own from the command line) |
| `icon.png` | Window icon of `sharptools.py` (optional, must be in the same folder) |

The `sharp_*.py` scripts also work on their own from the command line (see the end of this page), but you do not need that to use SharpTools.

## Using SharpTools

**Tokenize / Detokenize.** Choose *Detokenize* (binary → BASIC text) or *Tokenize* (BASIC text → binary), pick the input file and press **Run**. For detokenizing the pocket computer is detected automatically; for tokenizing choose the model (`PC-1211`, `PC-1251`, `PC-1360`, `PC-1403`, `PC-1500`, `PC-1600`, `PC-E500`, `PC-G850` …). The output name is filled in from the input name (same folder, other extension: `banana.bas` gives `banana.bin` and back). Change it if you like, or clear it to show the listing in the log instead.

**Renumber.** Choose the BASIC listing, the first line number and the step, then press **Renumber**. The output is `NAME_renum.bas` unless you change it. All jumps (`GOTO`, `GOSUB`, `THEN`, `ELSE`, `ON x GOTO/GOSUB 10,20,30`, `ON ERROR GOTO`, `RESTORE`, `RUN`, `RESUME`, `LIST`, …) follow the new numbers. Fixed names such as `GOTO "INTRO"`, the label in front of a line (`10 "A" PRINT`), strings, `REM` / `'` comments, `DATA` lines, computed jumps and hex numbers are not touched. A jump to a line that does not exist is left as it is and reported.

*Only a range of lines:* fill in **From line** and/or **To line** and only those lines get new numbers; all other lines keep theirs, and jumps to the renumbered lines follow, also from lines outside the range. The new numbers must fit between the line before and the line after the range, otherwise you get a message with the free numbers. If *First line number* is empty the range keeps its first line number.

*Merge two programs:* choose the second program under **Merge another program behind this one**. It is renumbered (from *First line number*, by default the next full hundred after the last line of the first program) and its jumps follow; the first program stays unchanged. A jump in the second program to a line of the first program stays as it is and is reported. With **Keep both numberings** nothing is renumbered, but no line number may exist in both programs. The result is `NAME_merged.bas`. A merge cannot be combined with a range.

**Line ends / EOF.** Choose a text file. **Add SUB (1A) at end of file**, **Toggle LF ⇄ CR LF**, **All → LF** and **All → CR LF** change the file in memory and show the result in the log; nothing is written until you press **Save**, which asks for confirmation and then overwrites the file. **Reload** discards the changes. A file with CR LF (also a mixed one) can be switched to LF with *Toggle* or *All → LF*.

**Tape files.** A `.tap` file is a tape recording: a 10-byte header, the program with a checksum byte after every block of 120 bytes, and an end mark. Choose *Strip*, pick the tape file and press **Run** to get the bare image (`NAME.bin`, or `NAME_img.bin` if the tape file is itself called `.bin`) that the *Tokenize / Detokenize* tab reads. The checksums are verified and wrong blocks are reported. *Check only* shows the header, the number of blocks and the result of the check without writing anything. *Wrap* does the reverse for a bare image and writes `NAME.tap`. Reading and checking works for the PC-1260 … PC-1475 and PC-1280/1360/1470/1475 tape layout (ID 70 … 73, verified on real files) and the PC-125x layout (ID 20, 21, 80); wrapping is only available for the first one, with the header `70 00 00 00 00 00 00 00 5F 14` taken from real files (on the command line `--header-from other.tap` copies the header of another tape file).

**Compare.** Choose program A and program B (a BASIC listing, a tokenized image or a tape file each; images are detected and decoded) and press **Compare**. Lines are matched by their line number: the log lists lines that are only in A, lines that are only in B and lines with the same number but different text. Tick *Ignore line numbers (renumbered program)* to compare a program with a renumbered copy of itself, or after lines were inserted or deleted: both programs are renumbered 1, 2, 3 … internally (jump targets follow) and compared line by line, and the original line numbers are shown. *Ignore spaces and case* helps when a hand-typed listing is compared with a detokenized one. Two images are also compared byte by byte (size, differing bytes with their offsets) before the listings are compared. For images the model is detected unless you choose one.

**Convert model.** Converts only what depends on the display of the pocket computer: the `POKE` statements into the display memory and the `CALL` that activates the display. The rest of the program is not touched and not checked (no keyword or screen check). Choose a listing (`.bas`) or an image (`.bin`, also a `.tap` tape file), the source model under **From** (a listing needs it, an image can leave it empty), the target under **To**, and press **Convert**. Only the pocket computers whose display addresses are known can be chosen: PC-1401/02, PC-140x/1460 (1403, 1403H, 1460), PC-1475, PC-125x and PC-126x. A two-line display (PC-126x, PC-1475) can only be converted to another two-line display, so choosing PC-126x as **From** selects PC-1475 as **To** and the other way round; the one-line models convert among each other.

- If the program has no `POKE` into the display memory and no activation `CALL`, only a message is shown and no file is written.
- A `POKE` into the display memory is moved to the address of the same line, display position and column on the target. The models have different address blocks (PC-1403 six, PC-1475 four, PC-125x and PC-126x two, PC-1401/02 two): where the bytes of one `POKE` are not continuous on the target, the `POKE` is split into several `POKE` statements.
- The `CALL` that activates the display (`&05A2` on the PC-1401/02, `&04B8` on the PC-1403, `&11E0` on the PC-125x) is changed to the call of the target. The PC-1475 and PC-126x need none: it is removed (a line that held only the call is deleted, or stays as `REM` if a jump goes to it). If the target needs a call and the program has none, a line with the call is added in front of the first display `POKE` (in the same line if no line number is free before it or it is jumped to).
- The log shows, for every changed statement, the source and the target statement, and lists what could not be changed (a `POKE` inside `IF … THEN`, bytes the target has no place for, `POKE` / `CALL` with an address in a variable). **The result has to be reviewed manually**; the log says so. Different graphics positions (PC-125x position 1, PC-1403 position 24) are reported.
- The output has the type of the input: a listing gives a listing (`NAME_pc125x.bas`), an image gives an image (`NAME_pc125x.bin`; it is read with the tokens of the source and written with the tokens of the target). On the command line: `sharp_convert.py input --to MODEL [output] [--from MODEL] [--listing FILE]`.

**Model menus.** The model menus of the other tabs show families: PC-121x, PC-125x, PC-126x, PC-1401/02, PC-140x/1460, PC-144x, PC-145x, PC-1350, PC-1360, PC-1475, PC-150x, PC-1600, PC-E5xx, PC-G8xx and PC-E2xx. A family stands for models with the same token table; PC-E5xx works like the PC-E500S. On the command line the single names (PC-1251, PC-1403, PC-E500 …) and the family names (quote names with a `/`) both work. *Convert model* lists only the five models with known display addresses.

**Minify / Tidy.** Choose *Minify* to save memory on the pocket computer. The options remove lines that hold only a comment (`REM …` or `' …`), comments at the end of a line (`:REM …`), and the spaces that are not needed (`PRINT A`, `FOR I=1 TO N` and the space between two words stay; spaces around `=`, `+`, `:` and commas go). A jump to a removed line is changed to the next line that stays; a comment line with a label (`10 "A" REM …`) is never removed, and neither is a comment right after `THEN` / `ELSE`. The log shows the size of the tokenized image before and after, for the chosen model (default `PC-1403`). *Tidy* makes a listing easier to read: a space after and before keywords (`FORI=1TO10` becomes `FOR I=1 TO 10`), spaces around `=` `<` `>`, one space after `:` `,` `;`. Strings, comments and `DATA` lines are not touched, and only spaces and comments ever change, never the program text. The output is `NAME_min.bas` or `NAME_tidy.bas`.

**Program info.** Choose a BASIC listing (`.bas`) or a tokenized image (`.bin`) and press **Analyze**. The log shows the number of lines, the size of the tokenized image in bytes (what the program needs in the pocket's memory, plus a little overhead), the variables used (numbers, strings, arrays), **dead jumps** (a `GOTO`/`GOSUB`/`THEN` to a line that does not exist, or to a label that is not defined), **lines nothing jumps to** (the first line and lines with a label such as `10 "A"` are entry points and left out; lines that follow a `GOTO`/`END`/`STOP`/`RETURN` and are never jumped to are marked as probably never run), and two lists: which line calls which, and each jump target with its callers. *Brief* leaves the long lists out. The model decides the image size; for a listing it is `PC-1403` unless you choose another, for a `.bin` image it is detected. Text in strings, `REM` / `'` comments and `DATA` lines are not analysed, computed jumps (`GOTO A*10`) are listed but not checked.

**Default folders.** The *Settings* menu sets a default input folder (where the Browse dialogs for input files start) and a default output folder (where output names are suggested and the Browse dialogs for output files start). Without a default output folder the output Browse dialogs start in the default input folder, and a suggested output name stays next to its input file. *Show default folders* and *Clear default folders* are in the same menu. The folders are remembered in `.sharptools.json` in your home folder.

**Log / Result.** Every tab has its own *Log / Result* pane below its controls. It shows what the tool of that tab did, including warnings (for example about damaged lines or missing jump targets), and it keeps its text when you change tabs. A result always appears in the tab that started the run, and *Clear log* clears only that tab.

**Files in all tabs.** Every input field has a drop-down list. A file you choose or type in one tab is offered in the input fields of all other tabs, newest first. The files that a tool writes (Detokenize, Renumber, Tape files, Minify / Tidy, Convert model, and *Save* in Line ends / EOF) are added to the same list after a successful run, so you can use them straight away as input for the next step, for example Strip a tape file, Detokenize the image, Minify the listing, then Convert it. A file written by *Tokenize* (a tokenized image) is not offered in the other tabs. The log says *Available as input file in all tabs* when this happened. The list is kept while the program runs.


## pc-1403 -> ML (BASIC to machine code compiler, plugin)

The tab **pc-1403 -> ML** compiles a PC-1403 / PC-1403H BASIC listing (`.bas`) to SC61860 machine code. It is a plugin.

Files: `sharp_b2ml.py` (compiler + built-in SC61860 assembler, Python 3, standard library only) and
`sharptools_b2ml.py` (the tab "pc-1403 -> ML"). Put both next to `sharptools.py`; `sharp_tok.py` is needed only to read tokenized `.bin` files.

    python3 sharp_b2ml.py prog.bas [-o NAME] [--org 0xF560] [--data 0xFA00] [--chunk 1024] 

Writes `NAMENoHdr.bin` (code only, no header), `NAMEXXXX.bin` (16-byte header + code; XXXX = start address in hex, e.g. `hanoiE6F0.bin`) and `NAME_load1.bas …` (BASIC loaders). `--asm` also writes `NAME.asm` (generated listing incl. runtime).
On the PC-1403: RUN loader 1, `NEW`, RUN loader 2 …, `NEW`, then `CALL <address>` (printed by the last loader).
For a PC-1403H use `--chunk 0` (a single loader). Loaders show a percent value after every two DATA lines.

### Binary files with and without header
- `NAMENoHdr.bin`: the machine code only (what the loaders poke into memory).
- `NAMEXXXX.bin`: a 16-byte header followed by the machine code. The `.tap` files are no longer written.

Header (ID 0x76, CSAVE M / CLOAD M of the PC-1403):

| Offset | Size | Content |
|---|---|---|
| 0x00 | 1 | file type `76` |
| 0x01–0x07 | 7 | name (max. 7 upper-case characters) in reverse order, padded with `00` |
| 0x08 | 1 | `00` |
| 0x09–0x0A | 2 | start address, big endian (high byte first) |
| 0x0B–0x0C | 2 | length − 1, big endian |
| 0x0D–0x0E | 2 | `00 00` |
| 0x0F | 1 | sum of the 30 nibbles of bytes 0x00–0x0E, modulo 256 |

Example (512 bytes named TEST at &8500): `76 54 53 45 54 00 00 00 00 85 00 01 FF 00 00 5C` (this example is reproduced exactly by the compiler's header routine).
The header layout follows the description supplied by the user; it was checked against that example, not on the calculator.
Earlier versions' tape files (ID 76, loaded with CLOAD M on a PC-1403) are replaced by these files.

### Number modes
`--mode auto` (default) compiles with 16-bit signed integers (2 bytes per variable, in the data page; `AND OR` bitwise, overflow wraps).
When the program needs real numbers (decimal numbers, `/`, `^`, `SQR SIN COS TAN ASN ACS ATN LOG LN EXP DEG DMS FACT HSN HCS HTN AHS AHC AHT TEN PI`,
`STR$`, `VAL`, `USING`) it switches to **mixed mode**: only the variables, arrays and expressions that need real numbers use real
numbers (10-digit BCD, 8 bytes, in the heap); everything else stays integer. `--mode int` forces integers only (`/` truncates,
real features give an error), `--mode real` makes everything real (largest), `--mode mixed` forces mixed mode.

How the compiler decides (per variable, over the whole program):
- a variable or array is **real** as soon as a real value is assigned to it anywhere. Real values are: decimal numbers (a number typed with a
  decimal point or exponent, even `1.0` or `1E3`, is real - use `F=1.0` to make `F` real on purpose), numbers outside -32768..32767, `/`, `^`,
  the math functions, real variables, `INT()` of a real value, `VAL()`, and arithmetic (`+ - *`, `ABS`, `SQU`, negation) with a real operand;
- a `FOR` variable is real if its start, limit or step is real, or the step is not a constant; `READ` targets are real if any `DATA` item is a decimal;
  `INPUT` into an integer variable accepts whole numbers only (make the variable real to accept decimals);
- comparisons, `AND OR NOT`, `PEEK`, `POKE`, `ON ... GOTO`, `WAIT`, array subscripts and `SGN` are integer operations; a real operand is converted
  (the fractional part is dropped for `AND OR NOT PEEK POKE` and subscripts; comparisons of real values compare as real numbers).
- **integer variables wrap at 16 bits** (32767 + 1 = -32768), exactly as in `--mode int`. The compiler warns when an integer variable is
  multiplied by itself (factorials, powers); write e.g. `F=1.0` to make it real. `INT(A/B)` is real (division gives a real), so its variable is real too.
- the value stack holds 11 real numbers (13 in `--mode real`); `A(1)..A(26)` without `DIM` are the variables `A..Z`: they are all integer or all real.

Example: a program with a loop in whole numbers and one `SQR`: integer part about 800 bytes + the real library for `SQR`; the same
program in `--mode real` has every variable and operation real.

### Size reduction
The generated assembler is optimised before it is assembled; every step is switched by a bit of `B2ML_OPT` (default 63):
1 unused runtime routines are left out, 2 repeated address loads become 3-byte calls of shared stubs, 4 repeated runs of
simple instructions become subroutines (nesting limited, never print/ROM calls), 8 multi-letter variables use the free
letter slots of the data page, 16 string temporaries are allocated only when used (one character = 2 bytes),
32 constants below 256 are loaded with a 5-byte form.
The compiler also computes the worst-case depth of the CPU call stack (14 bytes are guaranteed free, ROM routines
included) and falls back to less aggressive optimisation if it would be exceeded.
Example (Hanoi, 55 lines): unoptimised integer mode 8,376 bytes code + 610 bytes heap, now **4,623 + 252**
(`--mode int`; `--mode auto` uses mixed mode because of `BT/2`: 5,954 + 452; all-real 6.7 KB + 1.5 KB heap).
Mixed mode saves space only when integer code dominates: for programs that are mostly real numbers (trig, strings with `STR$`) `--mode real` can be
smaller, and the compiler prints a note when that is the case.
Use `--mode int` on the command line (the plugin always uses auto) for programs whose divisions are exact or may truncate.

#### How every change is verified (in a CPU simulator, NOT on a real PC-1403)
Reference compiler (frozen) and new compiler run 126 test programs, the examples and 800 random programs; display lines,
status, variables, memory, stack depth (static and measured), and writes outside the program's own memory must agree.
Deliberately broken compilers are caught by the same checks. Hardware behaviour of the ROM display/key routines and
the CLOAD M tape header (ID 76 confirmed) remain outside this verification.

### Language
* variables A–Z and two-character names (`AB`, `X1`); hex constants `&1F`; `+ - * / ^`, `= <> < > <= >=`, `AND OR`, unary minus, brackets
* functions with or without brackets: `PEEK ABS INT SGN SQU RND NOT` (`INT` truncates, as in the manual; `RND n` gives 1..n)
* real mode: `SQR EXP LN LOG TEN SIN COS TAN ASN ACS ATN DEG DMS FACT HSN HCS HTN AHS AHC AHT PI`; `DEGREE RADIAN GRAD` select the angle unit (default DEGREE; kept in the data page, not changed by `CLEAR`). `SIN(X)^2` means `(SIN X)^2`
* strings: `A$`…`Z$` hold 7 characters (longer values are cut, as on the PC-1403), `DIM N$(n)*m` makes an array of strings of m characters (default 16, m ≤ 80, one subscript, range checked); `+` joins (result ≤ 80 characters, else status 11), `= <> < > <= >=` compare, `LEFT$ RIGHT$ MID$(s,i[,n]) CHR$ LEN ASC`, `STR$ VAL` (these two make the program real mode), string constants in `DATA`/`READ`, `PRINT` of any string expression. Strings are length byte + characters in the heap; expressions use 4 temporary buffers of 80 characters (nesting deeper than 4 is an error). String functions are BASIC library routines like the math functions: about 2.6 KB for a small integer-mode string program
* arrays `DIM X(n)`, `DIM X(r,c)` (constant sizes 0–255, from 0, range checked); `A(1)`…`A(26)` are A…Z; `CLEAR` zeroes all variables
* keyboard: `INPUT ["prompt";|"prompt",] var[, …]` (as in the manual: `?` when there is no prompt, `;` keeps the prompt next to the typed text, `,` clears it at the first key, further variables are prompted with `?`, empty ENTER keeps the old value; keys ENTER, ←/DEL as backspace, SHIFT and SML work as on the machine; BRK ends the program), `INKEY$` (returns "" for ENTER, SHIFT, arrows…). Key codes come from the ROM key scan routine `0E68`, the character tables are the ROM's own (copied into the program, 258 bytes)
* `USING "mask"` / `USING` / `PRINT USING "mask"; items`: `#` digits (sign included), `.`, `^` scientific, `&` string field, mixed masks, extra digits truncated, overflow = status 7 (the program is then real mode)
* `AREAD v` is accepted and does nothing (warning): a compiled program has no start value
* `[LET] v=expr`, `IF expr THEN line | statements`, `GOTO`, `GOSUB`, `RETURN`, `ON expr GOTO/GOSUB n1,n2,…`, `END`/`STOP`
* `FOR v=a TO b [STEP s]` / `NEXT [v]` (real mode: any step expression; the body runs at least once)
* `DATA`, `READ`, `RESTORE [line]`, `POKE addr,v1[,v2…]`, `PEEK`, `CALL address` (your own machine code; keep internal RAM 28–37 and the data page free)
* `PRINT` / `PAUSE` with `"text"`, numbers, `CHR$(n)`, `;`; `WAIT n` shows PRINT output for n·1/59 s instead of waiting for a key
* Labels (`10 "A": CLEAR`, `20 "B" PRINT ...`, or a label alone on a line) are omitted: they only name the DEF key start of the BASIC program, a compiled program has a single entry point. `GOTO "A"` / `GOSUB "A"` with a label are not supported.
* `TRON TROFF RANDOM` do nothing, `BEEP n` sounds n beeps (buzzer bit of port C, switched exactly like the ROM does; `BEEP n,f,d` uses only n; `BEEP ON/OFF` unsupported), `LPRINT` is compiled as `PRINT` (display, with a warning), `REM`, `'`, several statements per line with `:`

#### Not supported yet (error with the line number)
`CHAIN`, `MEM`, real printer output for `LPRINT`, the commas/columns of `PRINT`, and all tape/printer/program-editing commands (`CLOAD CSAVE LIST RUN NEW …`, which cannot exist in a compiled program).
All planned stages are done. Not planned: `CHAIN`, `MEM`, printer output and tape commands. String commands like STR$, LEFT$, RIGHT$ and MID$.

### Accuracy of the math functions (real mode)
Written in the same BASIC dialect and compiled into the program when used. Measured against Python's `math` over random arguments: relative error ≤ 5·10⁻¹⁰ for `SQR EXP LN LOG SIN COS TAN ATN ASN ACS TEN HSN HCS HTN`, ≤ 2·10⁻⁹ for `AHS AHT FACT` (`FACT 69` accumulates rounding), i.e. the last one or two of the ten digits can differ from a real PC-1403. Exact cases come out exact (`SIN 30`, `COS 60`, `TAN 45`, `ATN 1`, `ASN 1`, `LOG 1000`, `2^10`, `FACT 10`). `FACT` needs 0…69; `TEN`/`EXP` overflow above ≈230.26 / 100.

### Status byte (`DATA+0x10`, default `&FA10`)
0 ok, 9 stack too small, 2 GOSUB overflow, 3 RETURN without GOSUB, 4 expression too deep, 5 subscript out of range, 6 out of DATA, 7 division by zero, 8 overflow, 11 string too long / bad string argument, 10 argument out of range (`SQR -1`, `LN 0`, `ASN 2`, `FACT 2.5` …). The program stops and returns to BASIC on an error.

### Machine layout
Default: code directly below the heap, heap directly below the data page `0xFA00` (variables A–Z in integer mode, stacks: 12 GOSUB levels, expression stack). Internal RAM 0F–37 is swapped with a copy on entry/exit so the BASIC interpreter state survives (same method as the chess engine). Real-mode programs are large (a program with `SIN COS SQR LN ^` is about 10 KB): the compiler warns when code + heap exceed what fits into the 8 KB RAM of a plain PC-1403 — use the PC-1403H.

### Tested / not tested
* Tested in the SC61860 emulator (strings in both modes: concatenation, LEFT$/MID$/RIGHT$, comparisons, DATA/READ, arrays, STR$/VAL, error cases): ≈1500 random integer expressions, the FP core against Python `Decimal`, real programs (FOR/IF/GOSUB/ON/arrays/DATA/READ/PRINT formats, error statuses), all math functions (random arguments, error cases, DEGREE/RADIAN/GRAD), restore of internal RAM; loaders tokenize and detokenize identically with `sharp_tok.py`.
* The key scan was checked against the real ROM: the scan routine `0E68` is the one the handbook describes (key code 0…0x55 in A, C=1 when a key is down) and the character tables copied into the program are byte-identical to the ROM's tables (extrom bank 08, offsets 0x240/0x296/0x2EC). The program itself was run with the emulator's keyboard simulation.
* More tests done on real hardware.

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
python3 sharp_tap.py info|strip|wrap input [output]                # tape files (.tap)
python3 sharp_comp.py A B [--renumbered] [--loose] [--model M]   # compare two programs
python3 sharp_convert.py input --to PC-1360 [output.bin] [--from M] [--force] [--strict] [--listing F.bas]   # convert to another model
python3 sharp_rem.py minify|tidy input.bas [output.bas] [--keep-rem-lines] [--keep-comments] [--keep-spaces] [--model M]
python3 sharp_info.py input.bas [--model PC-1403] [--brief]     # program info (also input.bin)
python3 sharp_b2ml.py prog.bas [-o NAME] [--org 0xF560] [--data 0xFA00] [--chunk 1024] [--mode auto|int|mixed|real] [--asm]   # BASIC -> machine code
```

- Without an output file `sharp_tok.py` writes next to the input with the other extension (`banana.bas` ⇄ `banana.bin`), `sharp_renum.py` writes `NAME_renum.bas` (`NAME_merged.bas` for `--merge`), and `sharp_eol.py` writes `NAME_sub.ext`, `NAME_lf.ext` or `NAME_crlf.ext` (`--inplace` changes the input itself). `_out` is added if a name would equal the input, so the input is never overwritten by accident. Use `-` as output to print the result.
- `sharp_tok.py`: `--model` is the only option. Example: `10 PRINT "HI"` and `20 GOTO 10` become `00 0A 06 DE 22 48 49 22 0D 00 14 04 C6 31 30 0D FF` for a PC-1403.
- `sharp_eol.py toggle`: a file with CR LF (or any CR, also mixed line ends) becomes LF, a file with LF only becomes CR LF. `lf` and `crlf` set the line ends explicitly. Binary files (with `00` bytes) are refused. `sub` does nothing if the file already ends with SUB.

## Credits and license

Tables and character codes follow **Pocket Tools** (`bas2img`, `wav2bin`) and the Sharp manuals. The PC-1500/1600 tables derive from **SharpDataExchange** by Martin Erzberger (Polyform Noncommercial 1.0.0), so this project is for **noncommercial use only**.

