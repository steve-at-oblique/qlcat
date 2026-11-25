# QLcat for Sinclair QL

QLcat is a native 68000 file viewer and text stream job for QDOS and SMSQ/E.
It reads a file in buffered chunks, normalizes line endings (`CR` and `CRLF`
to native QDOS `LF`), passes printable ASCII (`$20`–`$7E`), tabs (`$09`), and
Sinclair QL accented characters (`$80`–`$FF`), and strips control characters
so binary or executable files do not corrupt console windows.

QLcat uses fixed stack buffers with zero heap allocation, allowing it to
stream files of any size cleanly even on a stock 128K Sinclair QL.

## Install and run

Copy `qlcat` from the project root in binary mode to your drive. For example:

```basic
EW win1_qlcat;"win1_boot"
```

The standalone `qlcat` transfer file includes a 30-byte QDOS header specifying
file type 1 and 1024 bytes of data space.

By default, QLcat writes a leading newline and then the file contents to the
main BASIC job's channel `#1`. If a bare filename is given without a device
prefix, QLcat prepends the current `DATA_USE` directory when set.

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
- `-p` paginates output using the target window's detected height (`IOP.WENQ`), displaying `-- More --` at the bottom of each page. Press `SPACE` or `ENTER` to continue, or `CTRL-SPACE`, `Q`, or `ESC` to quit early. Pagination automatically disables if output is redirected to a non-window channel such as a file.
- `-p<N>` paginates every `<N>` lines (for example, `-p20`).
- `#n` redirects output to an open SuperBASIC channel `#n` (with an optional comma or space before the path). Passing an open channel directly to `EW` or `EXEC_W` (such as `EW win1_qlcat,#3;"win1_file"`) is also supported.

Combine options (such as `"-np"` or `"-np15"`) or pass them separately before or
after the filename. For a path containing spaces, enclose the path in double
quotes inside a single-quoted BASIC string, as in `EW win1_qlcat;'-n "WIN1_My File_txt"'`.

## Resident BASIC command

Transfer `qlcat_rext` as a raw binary and `qlcat` as a QDOS executable.
Load the extension once per boot with Toolkit II or SMSQ/E:

```basic
LRESPR win1_qlcat_rext
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

Copy `qlcat_asm` as plain text. In the monitor, assemble the executable binary
(replace `FLP1_` with your source/output drive):

```text
ASM 0 FLP1_
OUT FLP1_qlcat_bin
INCLUDE FLP1_qlcat_asm
```

From SuperBASIC, save the assembled binary (size `N` bytes) with the required
1024-byte data space:

```basic
a=RESPR(N)
LBYTES FLP1_qlcat_bin,a
SEXEC FLP1_qlcat,a,N,1024
```

To build the raw resident extension, assemble `qlcat_rext_asm` to `qlcat_rext`
and load it directly with `LRESPR`.

## License

BSD 2-Clause License. See `LICENSE` for details.
