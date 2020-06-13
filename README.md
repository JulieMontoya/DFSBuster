# DFSBuster
Simple Perl script to extract and inject files into Acorn 8271 DFS disk images

Working on another project which involves writing software for the BBC Micro, I decided it would be much simpler to do the bulk of the development work using an emulator.  This bypasses slow cassette and disk loading times and also potentially makes possible the use of more powerful tools on the host system, than the standard BASIC editor included with the target system.

DFSBuster is designed to take as its input a disk image file; and allows the directory structure of the disk to be examined and individual files extracted, and files to be added to the disk image.  This may then be mounted in the emulator, allowing the exchange of data between the target and host systems.

The latest version of DFSBuster incorporates the ability to detokenise a BBC BASIC program as saved by SAVE, as well as convert BBC line endings  (CR or LF, CR)  to standard Unix line endings  (LF).

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
