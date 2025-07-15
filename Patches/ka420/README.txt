wjm 12-feb-1999: KA420-ROM-PATCH V1.0
	These are some PATCHes to the boot ROMs of various 
        KA420/KA430-based VAXen (Vs3100/xx, uVAX3100/10). 
	Some of them have actually been tried out already ...


Background:
	Boot-ROMs for Vs3100 (and early uVAX3100 models) traditionally 
	do not allow for SCSI system (boot) disks >1 GB, because they 
	exclusively use 6-byte SCSI READ and WRITE commands. 

	This patch to ROM code (always the same, only with some offsets
	adapted to the different ROM & VMB versions) provides for 
	10-byte SCSI commands in case the LBN to be accessed is .ge. 2^21 .
	So it ought to allow booting and dumping from/to larger disks.

	In order to make room for the new code, I sacrifice 
	the very last portion of the debug code contained within 
	the boot driver (i.e. the code that writes out a screen 
	titled 'DKBTDRIVER halting ...' and then HALTs). 
	That last portion would dump 6 words of the VAX stack; 
	since that's where SP points in case of a HALT anyway, 
	no information is actually lost.
	And btw, I didn't see that screen displayed yet - I know it
	from TVBTDRIVER & MKBTDRIVER, which halt this way when you try 
	to boot from e.g. MUA0 or MKA100, and SCSI unit 1 is in fact 
	a disk drive ...
		

Files:
	KA42AROM13W.COM is for Vs3100/30 & /40 "KA42-A V1.3".

	KA42AROM16W.COM is for Vs3100/30 & /40 "KA42-A V1.6".	

	KA43AROM12W1.COM and KA43AROM12W2.COM, both applying
	the same patches, are for Vs3100/76 "KA43-A V1.2".

	KA41AROM14W1.COM and KA41AROM14W2.COM, also both having
	the same effect, are for uVAX 3100/10 "KA41-A V1.4".

	KA42BROM13W2.COM is for Vs3100/38 & /48 "KA42-B V1.3"
	(this patch happens to be identical to KA41AROM14W2).

	KA4ROM_READ.EXE reads the boot ROM from within VMS.
	Needs PFNMAP privilege. Usage:
		$ MCR dev:[dir]KA4ROM_READ outputfile

	KA4ROM_CKSUM.EXE checks, and optionally 'fixes', the checksum(s)
	in a disk copy of the boot ROM (in the format produced by KA4ROM_READ).

	KA4ROM_SPLIT.EXE splits a disk copy into two files
	that might be useful for actually 'burning' new ROMs.

	[.TOOLS] has sources and .OBJ for the KA4ROM_* utilities.
	Typically, there's no need to re-built the .EXE which I've
	found to work on VMS V5.3 and higher.


'Upgrade' procedure (No support, no warranty, no guarantees!):

    (1) Find out boot ROM version (so far I *believe* this to be uniquely
	specified by the 'short' id ala "KA42-A V1.3" which is displayed
	at power-on, and by ">>> T 50" (?)). Note that there's also 
	a longer id, which can be had from ">>> SHOW VER"; the latter
	does *not* have the same "Vm.n" in it.

    (2) Copy boot ROM to disk, using KA4ROM_READ.EXE . Best to stick to
	the naming convention exemplified by the PATCH command files ...

    (3) If the ROM version from (1) is one of the 'known' ones, apply
	the corresponding 'patch command file' via
		$ PATCH @KA%%%ROM%%W*
	(Note that the command files provided 'know' the file names:
	input is KA%%%ROM%%.BIN , output is KA%%%ROM%%W.BIN ).

	If you have an 'unknown' ROM version, look at KA41AROM14W1.COM ,
	and see if it can be adapted by fixing up the offsets in the lines
	marked "???". (I did so for KA41AROM14 and KA43AROM12). Once you
	have figured out the offsets to VMBVERS and DKBTDRIVER, you may
	wish to plug them into one of the *W or *W2 files which then might
	work also, as a sort of 'proof' that your DKBTDRIVER is in fact
	identical to a know one ...). There are some text strings that
	should be easy to locate in a full DUMP of the ROM contents ...

	Sure I'd like to know what you find out in this step!

	Do *not* proceed if fixing "???" lines doesn't work out completely!
	You boot ROM may support booting from big SCSI disks already, or
	have an 'unknown' (to me) boot driver ...

    (4) Once the PATCH has been successfully applied, run KA4ROM_CKSUM
	on the *output* file, and answer 'y' to the question posed ...
	This will update the ROM checksums.

    (5) Somehow 'burn a new boot ROM'. In the Vs3100s that I have looked at,
	this is actually 2 chips, each 64k*16 bit. The one closer to the
	'front' (i.e. memory connector side of the board) has the low 16 bits,
	the other one ('rear', i.e. ethernet connector) has the high 16 bits 
	of each 32-bit longword. Both 27C210[A] and 27C1024 chips have been
	seen, even as a mixed pair, so apparently they're all the same ...

    (6) Tell me about your experiences ...


Wolfgang J. Moeller, GWDG, D-37077 Goettingen, F.R.Germany, <moeller@gwdg.de>
