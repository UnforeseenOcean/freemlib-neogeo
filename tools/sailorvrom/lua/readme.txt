Sailor VROM (Lua version) | v0.20 by freem
================================================================================
Real coders would tell me to write this in C, but screw parsing text files in C.
(This script should work on Lua 5.1 and Lua 5.2; Lua 5.3 has not been tested.)
================================================================================
[Introduction]
Sailor VROM is a Neo-Geo V ROM/.PCM file builder.

No, this won't encode ADPCM samples for you.

ADPCM-A and ADPCM-B encoders are available on the Internet. The ADPCM-B encoder
is by ValleyBell and Fred/FRONT; the ADPCM-A encoder is by freem (that's me!).

/!!\ Important Note /!!\
When running this script on anything that isn't Windows, you may need to change
the line feeds of your ADPCM-A and ADPCM-B file lists. (For example, I needed
to change the line endings to LF on Linux.)

I currently do not know of a decent workaround (aside from running dos2unix on
the list files, but that requires dos2unix) for this issue.

================================================================================
[Sample Lists]
The sample lists that the program expects are simple text files, with one sound
path/filename per line.

The ADPCM-B sample list requires the sampling rate (between 1800Hz and 55500Hz)
as well each filename, being separated with a pipe character ('|').

An example of an ADPCM-B sample entry with a default sampling rate of 22050Hz:
example.pcmb|22050

================================================================================
[Usage]
lua svrom.lua (options)

[Options]
As of version 0.10, the program handles the command line differently.

Possible options you can pass to the program:

--pcma=(path to adpcm-a list)
Sets the ADPCM-A sample list. (required)

--pcmb=(path to adpcm-b list)
Sets the ADPCM-B sample list. Ignored if mode is set to cd. (optional)

--outname=(path to sound rom output file)
Sets the output path/filename for the V ROM/.PCM file. (optional)
(default "output.v" for cart, "output.pcm" for cd)

--samplelist=(path to sample list output file)
Sets the output path/filename for the sample list. (default "samples.inc")

--samplestart=(address for sample list to start)
Sets the start point for the sample list addresses. (default 0)
Can accept decimal and hex formats (both $0000 and 0x0000).

--mode=("cart" or "cd" (without the quotes))
Sets up the output type. (optional)
(default "cart")
 * CD mode will enforce the ignoring of ADPCM-B.
 * CD mode will eventually force an upper limit of 512KiB per .PCM file.

--slformat=("vasm", "tniasm", or "wla" (without the quotes))
Sets the sample list to output in a specific format:
 * vasm:   word	0x0000,0x00CE (vasm oldstyle syntax)
 * tniasm: dw	$0000,$00CE   (tniasm syntax)
 * wla:    .dw	$0000,$00CE   (WLA syntax)

================================================================================
[Notes]
* This program will pad your samples with 0x80 (silence) if necessary, if your
  original ADPCM encoding tool hasn't done so already. As this happens only
  during processing, the original sample files are untouched.

* The PCM-B list _only_ accepts sampling rates in Hertz. If you enter "44.1"
  expecting 44.1KHz (instead of 44100), don't be surprised when everything
  sounds wrong (unless you re-calculate the Delta-N value yourself).

================================================================================
[To-Do]
Many things.
* TEST OUTPUT!
  I still haven't tested the output, even after jumping to v0.11.
  If you aren't me and you see this message, you should probably panic.

  However, I did a quick spot check using the data from smkdan's ADPCM-A demo,
  and the file and sample addresses checked out. (Still needs system testing.)

  (2015/10/26 edit)
  I have managed to make a file that's 31.7KB (ends at $7F00) instead of 32KB
  even (ends at $8000). Time to bugcheck!

* Sample size checking (e.g. if something will be too big)
  Max size for ADPCM-A targeted WAV files = 4MiB (1MiB when converted)
  Max size for ADPCM-B targeted WAV files depends on sampling rate.

* Output size checking
 * .PCM files: max 512KiB
 * V ROMs: max 16MiB total (max configs: 4 x 4MiB [typical], 2 x 8MiB [ca.2002])

* Better sample boundary detection/fixing (rearranging samples).
  Currently, the program creates an unused empty space when the next sample will
  span a 1MB barrier. It's possible that the sample set being imported has a
  sample that could fit in said empty space...

================================================================================
[Future Options]
These might appear in a future version of the program.

--maxsize=(positive integer, "kilobytes" [actually kibibytes, value*1024])
Specifies the maximum size of a sound data output file.
(default: ????) (maximum on cart: 8192KiB (8MiB), maximum on cd: 512KiB)

================================================================================
[Output Configurations]
This is fun (not)

<Early Carts>
ADPCM-A and ADPCM-B in separate V ROMs

<PCM Chip Carts>
8192KiB (or more?) maximum per chip?

"On some Cartridge boards, V A20~V A22 can be used to select which of the 4
possible V ROMs to use" - NeoGeo Dev Wiki

<16 Megabytes>
* 4x4 MiB configuration
* 2x8 MiB configuration (only possible with NEO-PCM2?)

<NeoBitz PROGBITZ1>
Supposedly this board can support 32MiB (banked?) V ROMs, along with a more
conventional 16MiB. All of this without a NEO-PCM chip in sight, though the only
known game released on the board so far uses a single 8MiB chip.

<Neo-Geo CD>
Maximum File Size: 512KiB?
Banks: 2x512KiB (1 Megabyte total)

Additionally, .PAT files can be loaded to patch Z80 code. These files are used
to replace sound sample addresses when different .PCM files are used.

.PAT file entry format:
* [word] Patch Address ($0000-$FFFF)
* [word] Sample Start Address
* [word] Sample End Address
* [word] Loop Start Address
* [word] Loop End Address
