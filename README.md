# SRJV

**IMPORTANT about this fork** : tested ok on a JV-1080, but the same card in a JD-990 has issues with loops (clicky loops or glitchly noise).

Hints: WAV and SFZ from official cards (extracted with script in this repo) works well on the JD-990, only the custom ones have glitches. An the original wav do not seem to contain "smpl" metadata, so that would be a possible hint : findind a tool that remove the "smpl" metadata, and using root key and loop points only in the SFZ. If anyone finds a solution for JD-990, i'm interested.

## Notes from chab (macOs)

```
python3 runme.py -i input/v3 -o output/v3
```

- i recommend using versions in order to go back to earlier samples/fsz if needed (in each version, you may test different loop points, eq, gains, sample rate, etc.)
- puts samples/sfz in folder input/v1, v2, etc and "build" the bin into output/v1.bin, output/v2.bin, etc.

## Notes from chab on preparing samples/sfz (macOs)

### Tools :

- Audacity: https://www.audacityteam.org/
- Endless Wav: https://www.bjoernbojahr.de/endlesswav.html (fantastic app! thanks to Björn Bojahr)
- HexFiend (optional): https://hexfiend.com/

### 1. Preparing the samples :

- in Audacity: open sample wav file > set project sample rate to 32k (to obtain a smaller file) > convert to mono > trim (fade out the last few samples to avoid clicks), export in WAV 24bit PCM
- in Endless WAV: edit the loop with fades > set the root key > select the checkbox "truncate at end", save (my preferences: "Reduce chunks" is checked only)

This should make a WAV file including a "smpl" section describing the loop points and root key (you can find "smpl" with a hex editor. e.g. HexFiend)

### 2. Preparing the sfz :

- text editor: enter a ```<region>``` for each mapping

IMPORTANT: Apparently if the wav files contains a "smpl" section, 3 attributes are read from the "smpl" data and NOT from the sfz (pitch_keycenter, loop_start, loop_end) (search "dataChunkValid" in ROMImport.py)

example with a single region

```
<region>
sample=diva_supersaw_60.wav
lokey=0
hikey=127
pitch_keycenter=60
loop_start=0
loop_end=48857
```

Since my loop points and root key were saved in a "smpl" chunk (by Endless WAV), 3 attributes attibutes are actually useless here : pitch_keycenter, loop_start and loop_end (i kept them there anyway).

Example with multiple regions
(2 regions taken from "AP Str Ens A.sfz" extracted from vintage synth card (those samples do NOT contain "smpl" data, so loop/pitch_keycenter are taken from the sfz)

```
<region>
sample=arpstr841.wav
region_label=arpstr841.wav
amplitude=85.039
lokey=0
hikey=54
pitch_keycenter=58
tune=10.938
delay_samples=0
offset=0
loop_start=69
loop_end=29222
loop_type=alternate
looptune=0.0
loop_mode=loop_continuous

<region>
sample=arpstr842.wav
region_label=arpstr842.wav
amplitude=90.551
lokey=55
hikey=60
pitch_keycenter=65
tune=8.984
delay_samples=0
offset=0
loop_start=92
loop_end=20918
loop_type=alternate
looptune=0.0
loop_mode=loop_continuous
```



## Info

This toolkit should make it feasible to import custom samples into an SR-JV80 ROM image. To make full use of this, it's ideal you also have a way to burn the created images onto an SR-JV80 card. The recommendation is the ROMulator tool by Sector101:

https://www.sector101.co.uk/sr-jv-romulator.html

With enough further hacking, it is possible to load the imported multisamples as is into the RolandCloud VST's for testing.

In addition, experimental patch support is also included. As it is now, the format of the ROM suggests it usually has two different patch tables per card: what's seemingly required is the patch data that's directly compatible for, namely, the JV-80, the other known compatible patches being for the JV-2080 and, famously for SR-JV80-04, the JD-990. Currently the way patch importing works assumes any of these three models, though the creator does not guarantee the required SysEx for the JV-80 will be identical in the ROM due to inability to verify with such a synth.

UPDATE: 1/7/2023 Out of pressure for it, the creator also caved and included a wave extractor script that also puts in the effort of converting the tables into SFZ files. Use on official ROM's at your own risk: this is included solely in order to verify the data.

## Pre-requisites

* Python 3

## Additional Notes About Sample Importing

1. Total filesize needs to conform to the size of the ROM image, meaning ideally your sample set should be 23 MB in 24-bit PCM audio or less. The remaining space is needed for the definition tables and headers.

2. Due to the ROM's being organized in 1 MB blocks, a single sample cannot be larger than exactly 992KB after compression.

3. All imported audio needs to be in mono with a bitrate of 24-bit, or 32-bit linear PCM.

4. If you wish to have a reverse playback variation of a multisample without needing to import duplicate samples, make sure to have "REV" in the name of the alternate sfz file. Currently the way the script works is to import both versions at once and pick one as needed to avoid too much complexity.

5. There can be no more than 255 multisamples in a ROM, and each multisample is limited to 16 samples.

6. There is an optional "Brighten.py" script which can be used to fix higher frequencies of wav files in advance if you decide it sounds too muddy when imported. Resulting files will be placed in a "Brighter" subdirectory and must replace the original files in their original locations when ready.

## Usage

1. Compile all of the samples you wish to use, accompanied by an SFZ file for the sake of the multisample entries.

>1a. Each SFZ filename should preferably begin with "xxx-" where the x's are numbers, though anything else for the first four characters that organize them the way you wish for the ROM will do. The format also only allows up to twelve characters, so anything more than 16 characters (including the prefix) will be truncated! Additionally, the required opcodes for full use of these tools are "sample" and "hikey".

>1b. All samples in the SFZ files should point to WAV samples located in the same directory. As of November 30th, 2023, the 'smpl' chunk is no longer required, but it's still recommended if you want to save space on the ROM by avoiding duplicate sample table entries.

2. If you're particularly technically savvy, an additional but optional step is to edit the headers of the included Template ROM. This will be necessary if you wish to import a recognizable card other than that ID.

3. If you wish to import patches as well, make sure there is a SysEx bank dump of JV-2080, JD-990, and/or JV-80 presets included. These dumps should be stored in the following path relative to the samples to import: "Patches/2080.syx" for JV-2080 patches, "Patches/80.syx" for JV-80 patches, and "Patches/990.syx" for JD990 patches. Note that if you wish to import more patches than the user bank size of the model allows (64 for JV-80 and JD-990, 128 for JV-2080), you'll need to merge multiple SysEx files into one (the assumed maximum is 256).

4. It should now hopefully be as simple as running "runme.py".

>4a. One way is to run it without arguments via Python 3, in which case you will be prompted for input folder, output filename, the option to import patches, and an option for verbose output (particularly useful if some samples report DC offset).

>4b. The other method is to run it in your command line tool with arguments: "runme.py -i (input folder) -o (output file) (-p) (-v)

The resulting file for loading into a card via ROMulator should now be available for testing. If you're savvy enough to convert to SRX ROM's on your own as well, it's recommended you load an SRX conversion based on the additionally-created "Result.bin" into RolandCloud SRX-series VST's first in case of audio output issues, which is a very real possibility with DC offset in particular, knowing the nature of DPCM audio. If this is not an option, be sure to also check the output of the scripts, turning on verbose mode as needed in order to determine problematic samples.


## Footnotes

Credit goes to https://github.com/hackyourlife for the SR-JV80 Scrambling C code, and for the Java classes making it possible to feasibly encode in the FCE-DPCM format. Technically, said code was since converted into Python for the sake of portability, but credit is still due for using them as a basis.

Additional credit goes to https://archive.org/details/@jv80tinkerer for their jv80_wave_extractor script as a foundation for the new extractor script, even though some changes had to be made for a seamless process, particularly increasing the bit depth to 24-bit and avoiding duplicate samples to make reimporting easier.
