# DFSBuster
Simple Perl script to extract and inject files into Acorn 8271 DFS disk images

Working on another project which involves writing software for the BBC Micro, I decided it would be much simpler to do the bulk of the development work using an emulator.  This bypasses slow cassette and disk loading times and also potentially makes possible the use of more powerful tools on the host system, than the standard BASIC editor included with the target system.

DFSBuster is designed to take as its input a disk image file; and allows the directory structure of the disk to be examined and individual files extracted, and files to be added to the disk image.  This may then be mounted in the emulator, allowing the exchange of data between the target and host systems.

DFSBuster also incorporates the ability to detokenise a BBC BASIC program as saved by SAVE, and to convert BBC line endings  (CR or LF, CR)  to standard Unix line endings  (LF).  Simple heuristics are used to determine file types as follows:
* If the execution address is between &8000 and &80FF, the file is assumed to be a tokenised BASIC program  (this seems to be an entry point into the BASIC ROM which actually implements the NEW command)  and the option is offered to convert this to a text representation suitable for editing directly on the host system.
* If either the load or execution address is &0000, the file is assumed to be text  (`*SPOOL` output or similar)  and the option is offered to replace BBC line endings with the host system's default line endings.
* Otherwise the file is assumed to be data and written exactly as-is.

# Usage

`dfsls image_filename`

If invoked by the name `dfsls` then the program edits immediately once
the disc catalogue is displayed.

`dfsbuster image_filename`

Interactively extract files from the disc image.

`dfsbuster OPTIONS image_filename`

Various non-interactive ways to use the program, especially within scripts.

## Interactive Operation

If invoked from a terminal session with no options, just an image filename,
the program will enter an interactive mode.  A catalogue of the files on the
disc is displayed, with a number next to each one.  Entering a number at the
prompt will extract the contents of the appropriate file to the host system.
You will then be prompted for the host side file to write.

+ If the execution address of the file is between &8000 and &80FF, this could indicate a BASIC program; and the option will be presented to detokenise it, i.e. to render BASIC tokens as ASCII text.
+ If the load and/or execution address of the file is &0000, this could indicate a `*SPOOL` or similar text file; and the option will be presented to replace BBC Micro-style (CR or LF, CR) line endings with host system line endings.
+ Otherwise, the file contents will be passed through unchanged.

To exit, press ENTER without typing a file number.

## Non-Interactive Operation

If invoked with the option `-l`  (mnemonic: **l**ist),  then the program behaves as though it
had been invoked as `dfsls`.  The same behaviour will occur if either STDIN or STDOUT is
not a terminal.

If invoked with the option `-i`  (mnemonic: **i**nfo),  then the program produces output in an
identical format to the output of the DFS command `*INFO *.*`.  This may
be useful in Makefiles for multi-part assembler projects.

If invoked with the option `-1`  (mnemonic: **1**-column),  then the program displays just the
target-side filenames in a single column and exits.

If invoked with the option `-g`  (mnemonic: **g**rep)  and a filename pattern, then the program
displays the target-side filenames which match the pattern in one column
and the disc image name in another column, then exits.

(This output format should make it easy to do interesting things with `awk`.)

If invoked with the option `-x`  (mnemonic: e**x**tract)  and a filename pattern, then the
program extracts the contents of the first file matching the pattern.
There are some additional options available when using -x as folllows:

The `-o` option  (mnemonic: **o**utput_file)  additionally takes a filename, which will be
used on the host for the extracted file.  If -o is absent, then the DFS
filename will be used almost verbatim; files in directory **$** will have
the `$.` stripped, but files in any other directory will retain their
directory prefixes.  The special output filename **-** extracts to STDOUT.

The `-r`, `-t` and `-b` options  (mnemonics: **r**aw, **t**ext, **b**asic)  can be used to
force a particular translation mode, as folllows:
raw  (no translation), text  (replaces CR or LF, CR with LF, for
editing text files on the host system)  or BASIC  (detokenises
BASIC programs to text representation, like using `*SPOOL F.ILE`
followed by `LIST`).  If all are absent, then the translation mode will
be determined automatically based on the file's load and execution
addresses, as in interactive mode.

## History

DFS was the BBC Micro's Disk Filing System, and seems to be a descendant of the Acorn System DFS.  It uses a very simple disk structure.  Each track of the disk is divided into 10 sectors.  Each disk sector is 256 bytes in size, corresponding exactly to a machine page.  Sectors are numbered sequentially around one track and on to the next track*

The first two sectors  (=512 bytes)  of a disk are reserved for the Catalogue.  This contains one "disk" entry and up to 31 "file" entries.  Each entry occupies eight bytes in the first sector and eight bytes in the second sector.

* The disc title (12 characters) is stored in bytes 0-7 and 256-259.
* The number of write cycles is stored in byte 260, but as a binary-coded decimal number.
* The number of files on the disc is stored in the upper five bits of byte 261.
* Bits 4 and 5 of byte 262 store the boot mode  (q.v.)
* Bits 1 and 0 of byte 262 are the high-order bits of the number of sectors available for files and the catalogue.
* Byte 263 is the low-order bits of the number of sectors.

Files are not dated in any way  (this means the BBC Micro is, by happy accident, Y2038-compliant!)  but changes to the disk contents are tracked by means of a "write count".  This is a BCD number.

The boot mode may be 00 = not bootable, 01 = LOAD, 10 = RUN or 11 = EXEC.

The number of sectors is stored because some disk drives used with the BBC Micro supported 40 tracks; others supported 80 tracks, and therefore twice as much information before a disk was full.  This prevents you trying to step the head past the end of its travel and instead just end up overwriting the last track written.  If part of a disk is damaged, it's possible to format just as far as the damage and mark only the successfully-formatted sectors as usable.

* In practice, since a sector once read from disk has to be copied in memory which takes at least as long as reading it, but the disk carries on turning while the processor is doing its stuff, the sectors might actually be ordered 0, 7, 4, 1, 8, 5, 2, 9, 6, 3 around the disk; so the next sector to be read is just coming up to the head once the last sector has been dealt with.  Sectors are given their logical order number when the disk is formatted; that is then set in stone, and as far as DFS is concerned all references to sector numbers are logical, as opposed to physical, sectors.

NO RIGHTS RESERVED.  THIS CODE IS DEDICATED TO THE PUBLIC DOMAIN.

YOU MAY USE IT, ABUSE IT, ENJOY IT, DESTROY IT, STUDY IT, SHARE IT AND ADAPT IT WITHOUT RESTRICTION.
