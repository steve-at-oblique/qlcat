# QLcat for Sinclair QL

QLcat is a native 68000 file viewer and text stream job for QDOS and SMSQ/E.
It reads a file in buffered chunks, normalizes line endings (`CR` and `CRLF`
to native QDOS `LF`), passes printable ASCII (`$20`–`$7E`), tabs (`$09`), and
Sinclair QL accented characters (`$80`–`$FF`), and strips control characters
so binary or executable files do not corrupt console windows.

QLcat uses fixed stack buffers with zero heap allocation, allowing it to
stream files of any size cleanly even on a stock 128K Sinclair QL.

## Install and run

Transfer `qlcat` with a tool that reads the 30-byte QDOS transfer header and
restores the native executable metadata. For example, after transferring it to
`WIN1_`:

```basic
EW win1_qlcat;"win1_boot"
```

The standalone `qlcat` transfer file includes a 30-byte QDOS header specifying
file type 1 and 1024 bytes of data space.

By default, QLcat writes a leading newline and then the file contents to the
main BASIC job's channel `#1`. QLcat prepends the current `DATA_USE` directory
to relative names, including names such as `notes_txt`.

## Command-line options

With Toolkit II or SMSQ/E, pass options and a filename to `EW` or `EXEC_W`.

```basic
EW win1_qlcat;"win1_qlcat_asm"
EW win1_qlcat;"-n win1_qlcat_asm"
EW win1_qlcat;"-p win1_qlcat_asm"
EW win1_qlcat;"-np20 win1_qlcat_asm"
EW win1_qlcat;"#3,win1_qlcat_asm"
EW win1_qlcat,#3;"-n win1_qlcat_asm"
```

- `-n` prefixes each line with a right-aligned 5-digit line number and space.
- `-p` paginates output using the target window's character dimensions (`IOW.CHRQ`), displaying `-- More --` at the bottom of each page. Press `SPACE` or `ENTER` to continue, or `CTRL-SPACE`, `Q`, or `ESC` to quit early. Pagination automatically disables if output is redirected to a non-window channel such as a file.
- `-p<N>` paginates after every `<N>` displayed rows (for example, `-p20`).
- `#n` redirects output to an open SuperBASIC channel `#n` (with an optional comma or space before the path). Passing an open channel directly to `EW` or `EXEC_W` (such as `EW win1_qlcat,#3;"win1_file"`) is also supported.

Combine options (such as `"-np"` or `"-np15"`) or pass them separately before or
after the filename. For a path containing spaces, enclose the path in double
quotes inside a single-quoted BASIC string, as in `EW win1_qlcat;'-n "WIN1_My File_txt"'`.

## Resident BASIC command

Transfer `cat_rext` as a raw binary and `qlcat` as a QDOS executable.
Load the extension once per boot:

```basic
LRESPR win1_cat_rext
CAT_USE "win1_qlcat"
CAT win1_qlcat_asm
CAT win1_qlcat_asm,"-np"
CAT "-n","-p15",win1_qlcat_asm
```

`CAT_USE` sets the full filename of the `qlcat` executable (defaulting to
`win1_qlcat`), which can live on a different drive from the extension or the
file being displayed. Quote paths containing spaces and pass option flags as
quoted BASIC arguments separated by commas. The command waits for the listing
to finish.

## Build with Talent/Quanta Assembler Workbench 2.02

Transfer `qlcat_asm` and `qlcat_s` to the QL as plain text. These examples
use the `snl6_` drive prefix. Substitute your own drive as needed.
`cmd>` is the Workbench monitor prompt and `asm>` is the assembler prompt.

### Assemble the executable

```text
cmd> heap &3000
cmd> asm <start> snl6_
asm> in snl6_qlcat_asm
cmd> sexec snl6_qlcat <start> <end> &1024
```

Use the address printed by `heap` for `<start>` and the assembly end
address for `<end>`. The source's `END` directive returns you to the
monitor. `sexec` saves a native executable with 1024 bytes of data space.

### Assemble the resident extension

```text
cmd> heap &3000
cmd> asm <start> snl6_
asm> in snl6_qlcat_s
cmd> sbytes snl6_cat_rext <start> <end>
```

Use the addresses from the extension's own assembly. `sbytes` saves
`cat_rext` as a raw binary for `LRESPR`. On stock QDOS, reserve the
extension's size with `RESPR`, load it at that address with `LBYTES`,
and initialize it with `CALL`.

When copying the native executable back to the host, use a transfer tool
that exports it with the 30-byte QDOS header. If your tool exports only
the raw code, save that as `qlcat_bin` on the host and add the header
before shipping it as `qlcat`:

```sh
python3 -c 'from pathlib import Path; from struct import pack; Path("qlcat").write_bytes(b"]!QDOS File Header" + pack(">HBBII", 15, 0, 1, 1024, 0) + Path("qlcat_bin").read_bytes())'
```

## License

BSD 2-Clause License. See `LICENSE` for details.
