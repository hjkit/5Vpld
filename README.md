# Overview
This repository centers around modern Linux and Windows workflows for Atmel (Now Microchip) 5V GAL PLD and CPLD parts.

These parts are still active and highly worth considering wherever prototyping and 5V logic are a requirement. They can easily replace large numbers of TTL/CMOS logic gates and can be reprogrammed many times. Finally, the DIP parts are easy to solder, and the PLCC parts can be placed into a through-hole socket.

This is mostly documentation, but some small scripts are here that help make things easier and provide examples on how to avoid WinCUPL while still utilizing these parts.

This repository aims to make it easier to work with the following parts:
* GAL Devices: ATF16V8, ATF22V10 (Require an EPROM Programmer)
  * Part number convention seems to be: "number of inputs" V "number of outputs/macrocells".
* CPLD Devices (JTAG Programmable): ATF1502AS (32 macrocell), ATF1504AS (64 macrocell), ATF1508AS (128 macrocells)
  * The devices ending in AS and ASL are considered here. Both are active, however, ASL parts seem to be difficult to obtain.
  * Available in TQFP, PLCC, PQFP variants. PLCCs can be placed in through-hole sockets.

<details>
<summary>Scope: Expand here for parts not covered and why</summary>

* 3.3V parts are not considered: There are simply better choices that are well documented.
* The Greekpak devices probably should be covered here, but, they're reasonably well documented with modern tools.
* Any parts that are NRND or inactive. Because there are 5V parts that are still considered active, we only consider these.
* For the ATF150x CPLD parts specifically:
  * The BE and ASV devices not covered here as they seem to be difficult to obtain and are not 5V devices. If you need 3.3V or lower, there are probably better parts suited to your needs.
</details>

<details>
<summary>Expand here for a description of how these parts compare to ladder logic on a PLC</summary>

* Each rung's output in ladder-logic can be thought of as a single macrocell.
* The inputs on a rung can be "normally open" or "normally closed" (active high or low), and can consist of any number of inputs (or even the state of another macrocell). There can be combined in ways that are logical AND, OR, and so on.
</details>


<details>
<summary>Expand here for details on how all of these compare to FPGAs</summary>
Such parts are the spiritual predecessors of more modern FPGAs. Key differences between FPGAs and PLDs:

* FPGAs are typically constructed from a large number of LUTs (Lookup tables). CPLDs use a sum-of-products structure.
* FPGAs typically expect to have their bitstream uploaded on powerup, requiring an external EEPROM. PLDs are typically non-volatile.
* FPGAs usually support standard JTAG for programming, whereas many PLDs required specialized device programmers.
* There are likely exceptions to all of the above in some parts. These are not hard rules.
</details>

# Terminology / Background
<a href="https://en.wikipedia.org/wiki/Programmable_logic_device">PLD - </a>Programmable Logic Device<br />
<a href="https://en.wikipedia.org/wiki/Programmable_logic_device#GALs">GAL - </a>Generic Array Logic<br />
<a href="https://en.wikipedia.org/wiki/Programmable_logic_device#CPLDs">CPLD - </a>Complex Programmable Logic Device<br />

<a href="https://en.wikipedia.org/wiki/Macrocell_array">Macrocell</a> - Each output has a macrocell associated with it. These can often be configured as active high, active low, flip-flops, etc.<br />
Product Term - Each macrocell has a number of product terms associated with it (typically around 5). A product term is essentially a giant AND gate with inputs to each pin on the device. Burning away fuses allows selecting which inputs are fed into this AND gate, ultimately selecting the conditions required for a product term to be activated. Product terms belonging to the same output macrocell are then combined into an OR gate before being fed into the macrocell. This means that there can be several combination of inputs that allow a given macrocell to be triggered. This architecture is called a Sum-of-Products logic array.

<a href="https://en.wikipedia.org/wiki/Programmable_Array_Logic#CUPL">CUPL</a> - A early (1983) programming language used to define the behavior of digital logic gates. "Compiler for Universal Programmable Logic.", is essentially a predecessor to languages like Verilog/VHDL. CUPL.EXE is the compiler which is used to compile .PLD files written in CUPL, ultimately to be burned into programmable logic devices.<br />
<a href="https://www.microchip.com/en-us/products/fpgas-and-plds/spld-cplds/pld-design-resources">WinCUPL</a> - A Windows front-end/IDE to the CUPL compiler and related programs. It is still part specifically that we are trying to avoid, while keeping everything else underneath/around it as it is buggy.<br />

<a href="https://en.wikipedia.org/wiki/Netlist">Netlist</a> - A netlist is essentially an electrical schematic which defines connections. For the purposes here, it is an intermediary file format (Either EDIF or Berkeley PLA), which is used to describe the behavior of logic ultimately fed into the fitter.<br />
<a href="">.TT2</a> - The Berkeley PLA file format. An intermediary file which CUPL.EXE can generate that can be used by the Atmel fitters.<br />
<a href="https://en.wikipedia.org/wiki/EDIF">EDIF</a> - Another type of netlist format which is also usable by the Atmel fitters. Yosys is capable of generating this format, however, one will still need a techmap.<br />
<a href="https://en.wikipedia.org/wiki/Place_and_route">Fitter</a> - A fitter converts a netlist into the fusemap (.JED) file. Fitters are needed for the ATF150x CPLD devices. In more modern parlance, this is basically place & route.<br />
<a href="https://archive.org/details/JEDECJESD3C/mode/2up">.JED/JEDEC File</a> - A fuse map intended to be "burned/programmed" into a logic device.


.SVF File - Serial Vector Format. This file can be used by any JTAG programmer (vendor-independent) to program a device that has a JTAG interface.<br />
CSIM - A tool for simulating the behavior of logic. This takes an .SI file and produces an .SO file.


<a href="https://www.winehq.org/">Wine</a> - Wine is not an emulator. Allows running Windows programs under Linux.<br />


# Writing logic for these parts: Possible Workflows
Each of these subsections represents a potential workflow to design logic equations for these parts. The majority of the focus will be on methods that avoid WinCUPL (which is ultimately just an IDE/text-editor that calls CUPL.EXE).

This diagram is from the help files built into WinCUPL which shows how one can go from CUPL into the JED files needed to program a device.

Some of the other approaches covered here avoid CUPL entirely and instead generate netlists provided directly to the device fitter.

![WinCUPL Data Flow Diagram](vendor-docs/WinCUPL-data-flow-diagram.png)



## Old Approach: WinCUPL
While logic for these parts can be written via WinCUPL, the experience may be fraught with difficulty as it is somewhat unstable and requires Windows. It does however have value in the help files / documentation / examples. Furthermore, it should be noted that the CUPL compiler itself is actually pretty solid/stable. So, the recommended approach is to install and use it for documentation/examples and then simply avoid it for serious work by using the command line CUPL.EXE (perhaps through the helper scripts in this repository).

You can <a href="https://www.microchip.com/en-us/products/fpgas-and-plds/spld-cplds/pld-design-resources">Download WinCUPL from here</a>.

To get it working within Wine, you'll need winetricks so you can install mfc40 and mfc42. On Ubuntu Linux, this would look something like:
<code>sudo apt-get install wine winetricks playonlinux
winetricks mfc40 mfc42
</code>

Furthermore, if you are intending on working with the ATF150x parts, you should probably grab the newer fitters out of the Atmel Prochip package.

## Command line approach: CUPL & Your favorite text editor or IDE.
This is probably the most solid approach assuming you are OK with using CUPL as a language. This approach can operate on both Linux and Windows without trouble.
Since WinCUPL simply is a front-end / IDE on top of the CUPL.EXE compiler and related programs, one can write the desired logic in CUPL, save it in a .PLD file using their favorite editor and have CUPL.EXE compile it into a .JED file for programming into a PLD. CPLD parts will require the additional step of using a fitter for the specific device to produce the .JED file.

![A detailed User's Guide to CUPL in PDF](vendor-docs/CUPL_USERS_GUIDE.pdf)


Run CUPL using the following command line format:

<code>cupl [-flags] [library] [device] source</code>


Examples run under Wine would look like this:

<code>wine c:/Wincupl/Shared/cupl.exe -m1jn -u c:/Wincupl/Shared/cupl.dl your-code.PLD</code>

WARNING: Limit your the length of your filenames to 15 characters before the file extension (19 characters total) and do not allow multiple periods in the filename. Otherwise, CSIM.EXE seems to throw an error in the .SO file along the lines of <code>[0001sa] could not open:  terrible-long-fn....H.jed</code>

Additionally, if you are targeting a CPLD (ATF150x) for which CUPL.EXE does not have direct support, you will need to run:

<code>wine c:/Wincupl/WinCupl/Fitters/fit1502.exe -i your-code.tt2 -dev P1502T44 -DEBUG on -Verilog_sim VERILOG -Out_Edif ON</code>

The above example is for an ATF1502 in a TQFP-44 package. You will need to use the appropriate fitter and device type for your particular CPLD.

Additionally, you may want to explore the following environment variables:<br />
<code>FITTERDIR=C:\Wincupl\Wincupl\Fitters
LIBCUPL=c:\Wincupl\Shared\atmel.dl
</code>

<details>
<summary>Expand here for details of the command line flags for CUPL.EXE</summary>
Run CUPL using the following command line format:

<code>cupl [-flags] [library] [device] source
where
-flags is the following set of compiler options:
-j JEDEC download format
-h ASCII-HEX download format
-i HL download format
-n use input filename for output file
-a create absolute file
-l create listing file
-e create expanded macro definition file
-x create expanded product-terms in documentation file
-f create fuse plot/chip diagram in documentation file
-p create PDIF database interchange format file
-b create Berkeley PLA format file
-c create PALASM format file
-d deactivate unused OR terms
-r disable product term merging
-g program security fuse
-o treat all state machines as “one-hot”
-u use specified library for compilation
-s perform logic simulation after compilation
-w perform simulation with waveform output (MS-DOS only)
-m0 no minimization
-m1 quick minimization (default)
-m2 Quine McCluskey
-m3 Presto
-m4 Expresso
-q MIcrosoft format for error messages
-zq QuickLogic’s QDIF file
-kb Optimize product term usage for pin or pinnode variables. This overrides the DEMORGAN statement if it appears in the source file
-kd DeMorganize all pin and pinnode variables. This overrides the DEMORGAN statement if it appears in the source file
-ks Force product term sharing during minimization. This is also referred to as group reduction
-kx Do not expand XOR to AND-OR equations. This is used for device independent designs or designs targeted for fitter-supported devices where the fitter supports XOR gates
</code>
</details>

<details>
<summary>Expand here if you are interested in using VS Code as an IDE</summary>
 
Recently, two different extensions for VS Code for CUPL have been written:
* https://marketplace.visualstudio.com/items?itemName=tlgkccampbell.code-cupl
  * This one handles just syntax highlighting for CUPL .PLD files
* https://marketplace.visualstudio.com/items?itemName=VaynerSystems.VS-Cupl
  * This is an entire workflow, which has a bit more functionality beyond just syntax highlighting.
</details>

<details>
<summary>Expand here for a list of devices the version of CUPL provided with WinCUPL supports</summary>
 
* CBLD.EXE will allow you to see a list of devices that are supported within the CUPL.DL device library.
* WinCUPL is specifically restricted to Atmel devices, however, other versions of CUPL found elsewhere will likely have parts from a broader array of manufacturers.

<code>wine ./cbld.exe -l
CBLD(PM): CUPL Device Library Management Program
Version 5.0a
Copyright (c) 1983, 1998 Logical Devices, Inc.
C:\Wincupl\Shared\CUPL.DL  rev:DLIB-h

Device        Rev   Pins  Fuses  Pterms
------------  ---   ----  -----  ------
v750           03    24   14394    171
v750b          02    24   14435    171
v750c          02    24   14504    171
v750cext       02    24   14504    171
v750cextppk    02    24   14504    171
v750cppk       02    24   14504    171
v2500          07    40   71648    416
v2500b         04    40   71745    416
v2500c         04    40   71816    416
v2500cppk      04    40   71816    416
f1500          01    44   15360    320
f1500t         01    44   15360    320
f1500a         01    44   15360    320
f1500at        01    44   15360    320
f1502plcc44    01    44   15360    320
f1502ispplcc44   01    44   15360    320
f1502tqfp44    01    44   15360    320
f1502isptqfp44   01    44   15360    320
f1504plcc44    01    44   15360    320
f1504ispplcc44   01    44   15360    320
f1504tqfp44    01    44   15360    320
f1504isptqfp44   01    44   15360    320
f1504plcc68    02    68   15360    320
f1504ispplcc68   02    68   15360    320
f1504plcc84    02    84   15360    320
f1504ispplcc84   02    84   15360    320
f1504qfp100    02   100   15360    320
f1504ispqfp100   02   100   15360    320
f1504tqfp100   02   100   15360    320
f1504isptqfp100   02   100   15360    320
f1508plcc84    02    84   15360    320
f1508ispplcc84   02    84   15360    320
f1508qfp100    02   100   15360    320
f1508ispqfp100   02   100   15360    320
f1508tqfp100   02   100   15360    320
f1508isptqfp100   02   100   15360    320
f1508pqfp160   01   160   15360    320
f1508isppqfp160   01   160   15360    320
atfvirtual     01    44   99999   5000
g16v8          09    20    2194     64
g16v8ma        08    20    2194     64
g16v8ms        11    20    2194     64
g16v8a         03    20    2194     64
g16v8as        02    20    2194     64
g16v8s         09    20    2194     64
g16v8cpms      01    20    2195     64
g16v8cp        01    20    2195     64
g16v8cpas      01    20    2195     64
g16v8cpma      01    20    2195     64
g20v8          03    24    2706     64
g20v8ma        03    24    2706     64
g20v8ms        03    24    2706     64
g20v8a         02    24    2706     64
g20v8as        01    24    2706     64
g20v8cp        02    24    2707     64
g20v8cps       01    24    2707     64
g20v8cpma      03    24    2707     64
g20v8cpms      03    24    2707     64
g22v10         01    24    5892    132
g22v10cp       01    24    5893    132
p22v10         17    24    5828    132
virtual        01   200   99999   5000
v750lcc        07    28   14394    171
v750blcc       02    28   14435    171
v750clcc       02    28   14504    171
v750cextlcc    02    28   14504    171
v750cextppklcc   02    28   14504    171
v750cppklcc    02    28   14504    171
v2500lcc       08    44   71648    416
v2500blcc      04    44   71745    416
v2500clcc      04    44   71816    416
v2500cppklcc   04    44   71816    416
g20v8lcc       03    28    2706     64
g20v8alcc      02    28    2706     64
g20v8aslcc     01    28    2706     64
g20v8malcc     03    28    2706     64
g20v8mslcc     03    28    2706     64
g20v8slcc      03    28    2706     64
g20v8cplcc     02    28    2707     64
g20v8cpslcc    01    28    2707     64
g20v8cpmalcc   03    28    2707     64
g20v8cpmslcc   03    28    2707     64
g22v10lcc      02    28    5892    132
g22v10cplcc    01    28    5893    132
p22v10lcc      17    28    5828    132
</code>
</details>


If one is trying to utilize the ATF150X devices, using the appropriate fitter is required.
* ![ATF15xx Family Device Fitter User's Manual](vendor-docs/fitter.pdf)

<details>
<summary>Expand for command line options for the latest known version of the ATF1502.EXE fitter.</summary>
<code>Atmel ATF1502 Fitter Version 1918 (3-21-07)
Copyright 1999,2000 Atmel Corporation
 Usage: FIT1502.EXE [-i] input_file[.tt2] {options}
 Options:
   -help
   -o output_file_name (for *.tt3 and *.jed)
   -device package_type (PLCC44/TQFP44)
   -tech tech_name (ATF1502AS/ATF1502ASV/ATF1502BE)
   -module module_name
   -preassign TRY|keep|ignore (pin preassignment options)
   -silent (no message on screen)
   -h2 (advanced help option)
   -has (advanced help option for AS)
   -hbe (advanced help option for BE)
</code>

Advanced help options:
<code>
Atmel ATF1502 Fitter Version 1918 (3-21-07)
Copyright 1999,2000 Atmel Corporation
   -strategy c [command file name]
   -strategy ifmt (input file format) [TT | edif]
   -strategy lib (library file name for edif input)
   -strategy open_collector = [   OFF |   on  | = pin_name1 pin_name2...]
   -strategy JTAG = [   off |   ON ]
   -strategy pd1 [   OFF |   on ] (power down 1)
   -strategy pd2 [   OFF |   on ] (power down 2)
   -strategy TDI_pullup = [   OFF |   on ]
   -strategy TMS_pullup = [   OFF |   on ]
   -strategy DEBUG = [   on |   OFF ]
   -strategy output_fast [on | OFF | = pin_name1 pin_name2...]
   -strategy pin_keep [ off | = pin_name1 pin_name2...]
   -strategy ues [value ] (2 ASCII characters)
   -strategy security [ OFF | on ]
   -strategy tPD = [ 5 | 7 ]
   -strategy voltage_level_A [ 1.8 | 2.5 | 3.3]
   -strategy voltage_level_B [ 1.8 | 2.5 | 3.3]
   -strategy fast_inlatch [ OFF | on | = pin_name1 pin_name2...]
   -strategy schmitt_trigger [ OFF | = pin_name1 pin_name2...]
   -strategy pull_up [ OFF | = pin_name1 pin_name2...]
   -strategy unused_To_PinKeeper [ off | ON ]
   -strategy pull_up_unused [ OFF | on]
   -strategy unused_To_Ground [ OFF | on]
   -strategy pull_down [ OFF | = dedicated_pin1 dedicated_pin2...]
   -strategy Latch_Synthesis [ON | off ]
   -strategy Optimize [ON | off]
   -strategy Cascade_Logic [ON | off |= pin_name1 ..pin_nameN]
   -strategy Foldback_Logic [ON | off |= node_name1 ..node_nameN]
   -strategy Soft_Buffer [on | OFF |= node_name1 ..node_nameN]
   -strategy XOR_Synthesis [on | OFF |= pin_name1 ..pin_nameN]
   -strategy Push_Gate [on | OFF]
   -strategy Verilog_sim [sdf | Verilog | OFF]
   -strategy Vhdl_sim [sdf | vhdl | OFF]
   -strategy Out_Edif [on | OFF]
   -strategy Global_Fold [node_name1 ..node_nameN]
   -strategy Global_OE [node_name1 ..node_nameN]
   -strategy OE_node [node_Number1..node_NumberN]
   -strategy logic_doubling [on | OFF]
   -strategy twoclock [clockname]
   -strategy pinfile
</code>
</details>

Other people's workflows:
* https://github.com/willie68/WCPLD
* https://github.com/Manawyrm/PAL-GAL-CI

## Absurd approach: Fusemaps by hand
* See this <a href="https://blog.frankdecaire.com/2017/01/22/generic-array-logic-devices/">blog post</a> by Frank DeCaire.


While not the easiest approach, just as one can write G-Code in notepad or Assembly code in a hex editor, manually creating a fusemap is technically possible. This assumes that you have a datasheet for your device which has a description of the fusemap and the details of how the macrocells work. With this in hand, one could write a JEDEC file with the desired functionality and a text editor. This would be non-trivial and error-prone, but it demonstrates that such a thing could be done, at least with the older PLDs (16V8, 22V10), and even with the ATF750 (some datasheets actually had the fusemap for this part).


It is worth noting that the fusemap for the ATF150x parts has been recently documented in <a href="https://github.com/whitequark/prjbureau">prjbureau</a>. Given the complexity of these devices over PLDs, writing a fusemap by hand for these parts would probably be a bad idea.

## Other languages: ABEL, PALASM (ancient)
Since we're mostly covering modern approaches to these devices here, these will only be covered very briefly:
* ABEL: "Advanced Boolean Expression Language" was created in 1983 by Data I/O Corporation.
* PALASM: Introduced by Monolithic Memories, Inc. (MMI) in the 1980's
  * A modern version of this is called <a href="https://github.com/daveho/GALasm">GALASM</a> which is a continuation of something called GALer. This might be worth considering if you are happy with just PLDs.

## Atmel Prochip (Not Free, Verilog/VHDL support)
![PDF: Example Verilog Design flows with using ProChip 5.0.1](vendor-docs/CPLD_Mentor_Verilog_tutorial[1].pdf)<br />
Atmel Prochip is not free, however, you can <a href="https://ww1.microchip.com/downloads/en/DeviceDoc/ProChip5.0.1.zip">download it from here</a>, and may be able to <a href="https://www.microchip.com/prochiplicensing/#/">request a trial license from Microchip</a>. This workflow supports Verilog/VHDL, which is great if one wants to move away from CUPL entirely and can afford to purchase a license.

This packages is worth downloading regardless because there are newer fitters for the ATF150x devices that can be extracted from this installation, and these fitters are required in every other approach mentioned here. The newer versions of the fitters should mention version 1918 (3-21-07) when invoked from a command line. (The fitters that come with WinCUPL are old and should be replaced with the ones from this package.
## Quartus (Free, Verilog, VHDL, Schematic Capture). Indirect support for ATF150x.
* It turns out that the Altera (Now Intel) <a href="https://www.intel.com/content/www/us/en/software-kit/711791/intel-quartus-ii-web-edition-design-software-version-13-0sp1-for-windows.html?">Quartus 13.0sp1</a> can be used to produce a .POF file targeting various CPLD chips made by Altera in the MAX EPM3K/EPM7K series, which can be converted to target an ATF150x device.
* The resulting .POF file can be converted using a utility called <a href="http://ww1.microchip.com/downloads/archive/pof2jed.zip">POF2JED</a> from Atmel (Now Microchip). This is further detailed in <a href="http://ww1.microchip.com/downloads/en/AppNotes/DOC0916.PDF">this application note.
* Important!: Newer versions of Quartus will not work. v13.0sp1 last version that had support for the MAX EPM3K/EPM7K chips. Support for these chips has been removed from newer versions of Quartus. You MUST use the old version.

## Digital (free, use schematics instead of logic equations / programming)
"Digital is an easy-to-use digital logic designer and circuit simulator designed for educational purposes." This is an interesting option as one can create a schematic and have a .JED file generated for a GAL16V8 or GAL22V10. If one provides the fitters to Digital, it can produce .JED files for the ATF150x series as well.
https://github.com/hneemann/Digital

## Yosys (Open Source with Atmel Fitters, experimental)
In theory, one can use Yosys Open SYnthesis Suite (Yosys) with the help of the Atmel Fitters a specific CPLD and a techmap to produce .JED files. This is a bit more experimental, but some have managed to make this work. This allows an almost entirely open-source workflow using Verilog, and probably <a href="https://icestudio.io/">Icestudio</a> if one prefers schematic capture as well. A good place to start would be using the <a href="https://github.com/YosysHQ/oss-cad-suite-build">OSS CAD Suite</a> to get the big parts of the suite set up. After that, there are two approaches to making this work:
* https://github.com/whitequark/prjbureau
  * prjbureau demonstrates going from RTLIL to a .JED file
* https://github.com/hoglet67/atf15xx_yosys/
  * This example goes from plain old verilog into a .JED file by implementing a techmap.

# Programming / Burning and Device Information
There are a few choices on how the part can actually be programmed depending on whether it supports JTAG.

![A detailed overview of ways to program a given device.](PROGRAMMING.md)


# Reversing a JED file back into logic equations
Finally, if one is able to read a .JED out of a device, this can sometimes be reversed back into equations. These devices all have security fuses, however, which can disable any ability to read out the device. Given a .JED file, the following approaches can be taken to arrive at the equations:
* By hand, comparing the .JED file to the fusemap / macrocells in the datasheet. See this <a href="https://blog.frankdecaire.com/2017/01/22/generic-array-logic-devices/">blog post</a> by Frank DeCaire.
* JED2EQN.EXE - A DOS utility floating around on the internet.
* MAME can be compiled with a utility called jedutil

# Simulation
CSIM.EXE can be fed test vectors and be used to simulate the behavior of a particular chip, or even a virtual device. The following things are required to do this successfully:
* You must create an .SI file containing the desired test vectors.
* Provide an "Absolute file". The .ABS file is generated by CUPL.EXE when the -a flag is passed to it. This generates a binary file based on the logic equations from the source CUPL .PLD file.
CSIM will then generate a .SO output file, and optionally append test vectors to an existing .JED file for testing purposes.
<details>
<summary>Expand here for command-line options to CSIM.EXE</summary>
<code>csim [-flags] [library] [device] source
where
-flags is the following set of simulator options:
-l create listing file.
-j append test vectors to JEDEC file.
-n use source filename for JEDEC file.
-v display simulation results to terminal.
-u use specified library for simulation.
library is the library name and path name if the -u flag is being used to specify a
library other than the default library.
device must be the same device mnemonic as was used in the CUPL compilation.
Specifying the device is optional; if a device is not specified, CSIM uses the device
CUPL compiled (contained in the .ABS file).
source is the user-created ASCII test specification file (filename.SI). The
extension .SI is assumed for the source file and may be omitted when giving the
CSIM command.</code>
</details>


Creating a .SI file:<br />
* An .SI file should have the same header information as the original .PLD source file. If not, this will generate warnings.
* Comments begin with a /* and end with a */
* An .SI file can have the following keywords/statements: ORDER, BASE, and VECTORS
  * The ORDER keyword is used to list the variable / inputs and outputs to be used in the simulation table, and to define how they are displayed. Typically, the variable names are the same as those in the corresponding CUPL logic description file.
  * The BASE keyword specifies a number base. Hexadecimal is the default if unspecified.
  * The VECTORS keyboard specifies a list of test vectors (signals that are applied and expected outputs).
* If you simply want to see what will happen on the outputs rather than setting a pre-determined expected value, set the outputs to *

<details>
<summary>Expand for a list of valid Test Values used in a test vector</summary>
<code>0 Drive input LO (0 volts) (negate active-HI input)
1 Drive input HI (+5 volts) (assert active-HI input)
C Drive (clock) input LO, HI, LO
K Drive (clock) input HI, LO, HI
L Test output LO (0 volts) (active-HI output negated)
H Test output HI (+5 volts) (active-HI output asserted)
Z Test output for high impedance
X Input HI or LO, output HI or LO Note: Not all device programmers treat X on inputs the same; some put it to 0, some allow input to be pulled to 1, and some leave it at the previous value.
N Output not tested
P Preload internal registers (value is applied to !Q output)
* Outputs only -simulator determines test value and substitutes in vector
' ' Enclose input values to be expanded to a specified BASE (octal, decimal, or hex). Valid values are 0-F and X.
“ ” Enclose output values to be expanded to a specified BASE (octal, decimal, or hex.) Valid values are 0-F, H, L, Z, and X.
</code>
</details>
