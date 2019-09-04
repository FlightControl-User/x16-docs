# Commander X16 Programmer's Reference Guide



## BASIC Programming

### Commodore 64 Compatibility

The Commander X16 BASIC interpreter is 100% backwards-compatible with the Commodore 64 one. This includes the following features:

* All statements and functions
* Strings, arrays, integers, floats
* Max. 80 character BASIC lines
* Printing "quote mode" control characters like cursor control and color codes, e.g.:
	* `CHR$(147)`: clear screen
	* `CHR$(5)`: white text
	* `CHR$(18)`: reverse
	* `CHR$(14)`: switch to upper/lowercase font

Because of the differences in hardware, the following functions and statements are incompatible between C64 and X16 BASIC programs.

* `POKE`: write to a memory address
* `PEEK`: read from a memory address
* `WAIT`: wait for memory contents
* `SYS`: execute machine language code

The BASIC interpreter also currently shares all problems of the C64 version, like the slow garbage collector.

### New Statements and Functions

There are several new statement and functions. Note that all BASIC keywords (such as `FOR`) get converted into tokens (such as `$81`), and the tokens for the new keywords have not been finalized yet. Therefore, loading BASIC program saved from a different revision of BASIC may mix up keywords.

#### DOS

**TYPE: Command**
**FORMAT: DOS &lt;string&gt;**	

**Action:** This command works with the command/status channel or the directory of a Commodore DOS device and has different functionality depending on the type of argument.

* Without an argument, `DOS` prints the status string of the current device.
* With a string argument of `"8"` or `"9"`, it switches the current device to the given number.
* With an argument starting with `"$"`, it shows the directory of the device.
* Any other argument will be sent as a DOS command. 

**EXAMPLES of DOS Statement:**

      DOS"$" : REM SHOWS DIRECTORY
      DOS"S:BAD_FILE" : REM DELETES "BAD_FILE"
      DOS : PRINTS DOS STATUS, E.G. "01,FILES SCRATCHED,01,00"

#### MON

**TYPE: Command**
**FORMAT: MON**

**Action:** This command enters the machine language monitor. See the dedicated chapter for a  description.

**EXAMPLE of MON Statement:**

      MON

#### VPEEK

**TYPE: Integer Function**
**FORMAT: VPEEK (&lt;bank&gt;, &lt;address&gt;)**

**Action:** Return a byte from the video address space. The video address space has 20 bit addresses, which is exposed as 16 banks of 65536 addresses each.

**EXAMPLE of VPEEK Statement:**

      PRINT (PEEK(4,0) AND $E0) / 32 : REM PRINTS THE CURRENT MODE (0-7)

#### VPOKE

**TYPE: Command**
**FORMAT: VPOKE &lt;bank&gt;, &lt;address&gt;, &lt;value&gt;**

**Action:** Set a byte in the video address space. The video address space has 20 bit addresses, which is exposed as 16 banks of 65536 addresses each.

**EXAMPLE of VPOKE Statement:**

      POKE 0,1,1 * 16 + 2 : REM SETS THE COLORS OF THE CHARACTER
      REM AT 0/0 TO RED ON WHITE

### Other New Features

The numeric constants parser supports both hex (`$`) and binary (`%`) literals, like this:

      PRINT $EA31 + %1010

The size of hex and binary values is only restricted by the range that can be represented by BASIC's internal floating point representation.

## KERNAL

The Commander X16 contains a version of KERNAL as its operating system in ROM. It contains

* a 40/80 character screen driver
* a PS/2 keyboard driver
* an NES/SNES controller driver
* a Commodore Serial Bus ("IEC") driver *[not yet working]*
* an RS-232 driver *[not yet working]*
* "Channel I/O" for abstracting devices
* simple memory management
* timekeeping

### Commodore 64 API Compatibility

The KERNAL fully supports the C64 KERNAL API.

**Channel I/O:**
$FF90: `SETMSG` – set verbosity
$FFB7: `READST` – return status byte
$FFBA: `SETLFS` – set LA, FA and SA
$FFBD: `SETNAM` – set filename
$FFC0: `OPEN` – open a channel
$FFC3: `CLOSE` – close a channel
$FFC6: `CHKIN` – set channel for character input
$FFC9: `CHKOUT` – set channel for character output
$FFCC: `CLRCHN` – restore character I/O to screen/keyboard
$FFCF: `BASIN` – get character
$FFD2: `BSOUT` – write character
$FFD5: `LOAD` – load a file into memory
$FFD8: `SAVE` – save a file from memory
$FFE7: `CLALL` – close all channels

**Commodore Peripheral Bus:**
$FFB4: `TALK` – send TALK command
$FFB1: `LISTEN` – send LISTEN command
$FFAE: `UNLSN` – send UNLISTEN command
$FFAB: `UNTLK` – send UNTALK command
$FFA8: `IECOUT` – send byte to serial bus
$FFA5: `IECIN` – read byte from serial bus
$FFA2: `SETTMO` – set timeout
$FF96: `TKSA` – send TALK secondary address
$FF93: `SECOND` – send LISTEN secondary address

**Memory:**
$FF9C: `MEMBOT` – read/write address of start of usable RAM
$FF99: `MEMTOP` – read/write address of end of usable RAM

**Time:**
$FFDE: `RDTIM` – read system clock
$FFDB: `SETTIM` – write system clock
$FFEA: `UDTIM` – advance clock

**Other:**
$FFE1: `STOP` – test for STOP key
$FFE4: `GETIN` – get character from keyboard
$FFED: `SCREEN` – get the screen resolution
$FFF0: `PLOT` – read/write cursor position
$FFF3: `IOBASE` – return start of I/O area

Some notes:

* The Commodore Peripheral Bus calls first talk to the "Computer DOS" built into the ROM to detect an SD card, before falling back to the Commodore Serial Bus.
* The `IOBASE` call returns $9F60, the location of the first VIA controller.
* The `SETTMO` call has been a no-op since the Commodore VIC-20, and has no function on the X16 either.
* C64 compatibility extends into the layout of the zero page ($0000-$00FF) and, the KERNAL/BASIC variable space ($0200 -$02FF) and the vectors ($0300-$0333).

### Commodore 128 API Compatibility

In addition, the X16 supports a subset of the C128 API additions:

$FF4A: `CLOSE_ALL` – close all files on a device
$FF53: `BOOT_CALL` – boot load program from disk *[not yet implemented]*
$FF8D: `LKUPLA` – search tables for given LA
$FF8A: `LKUPSA` – search tables for given SA
$FF5F: `SWAPPER` – switch between 40 and 80 columns
$FF65: `PFKEY` – program a function key *[not yet implemented]*
$FF74: `FETCH` – LDA (fetvec),Y from any bank
$FF77: `STASH` – STA (stavec),Y to any bank
$FF7A: `CMPARE` – CMP (cmpvec),Y to any bank
$FF7D: `PRIMM` – print string following the caller’s code

One note:

* For `SWAPPER`, the user can detect the current mode by reading the zero page location `LLEN` ($D9), which either holds a value of 40 or 80. This is different than on the C128.

### New API for the Commander X16

There are a few new APIs. Please note that their addresses and their behavior is still prelimiary and can change between revisions.

$FF00: `MONITOR` – enter montior
$FF06: `GETJOY` – query joysticks
$FF6E: `JSRFAR` – gosub in another bank

#### Function Name: GETJOY

Purpose: Query the joysticks and store their state in the zeropage
Call address: $FF06 (hex) 65286 (decimal)
Communication registers: None
Preparatory routines: None
Error returns: None
Stack requirements: 0
Registers affected: .A, .X, .Y

**Description:** The routine `GETJOY` restrieves all state from the two joysticks and stores it in the zeropage locations `JOY1` and `JOY2`.

Each of these symbols consist of 3 bytes with the following layout:

      byte 0:      | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
              NES  | A | B |SEL|STA|UP |DN |LT |RT |
              SNES | B | Y |SEL|STA|UP |DN |LT |RT |
      
      byte 1:      | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
              NES  | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
              SNES | A | X | L | R | 1 | 1 | 1 | 1 |
      byte 2:
              $00 = joystick present
              $FF = joystick not present

* Presence can be detected by checking byte 2.
* NES vs. SNES can be detected by checking bits 0-3 in byte 1.
* If a button is pressed, the corresponding bit is zero.
* Note that bits 6 and 7 in byte 0 map to different buttons on NES and SNES.

**How to Use:**
1) Call this routine.
2) Read joystick state from `JOY1` and `JOY2`.

**EXAMPLE:**

      JSR GETJOY
      LDA JOY1
      AND #128
      BEQ NES_A_PRESSED

#### Function Name: JSRFAR

Purpose: Execute a routine on another RAM or ROM bank
Call address: $FF6E (hex) 65390 (decimal)
Communication registers: None
Preparatory routines: None
Error returns: None
Stack requirements: 4
Registers affected: None

**Description:** The routine `JSRFAR` enables code to execute some other code located on a specific RAM or ROM bank. This works independently of which RAM or ROM bank the currently executing code is residing in.
The 16 bit address and the 8 bit bank number have to follow the instruction stream. The `JSRFAR` routine will switch both the ROM and the RAM bank to the specified bank and restore it after the routine's `RTS`. Execution resumes after the 3 byte arguments.

**How to Use:**
1) Call this routine.

**EXAMPLE:**

      JSR JSRFAR
      .WORD $C000 ; ADDRESS
      .BYTE 1     ; BANK

#### Function Name: MONITOR

Purpose: Enter the machine language monitor
Call address: $FF00 (hex) 65280 (decimal)
Communication registers: None
Preparatory routines: None
Error returns: Does not return
Stack requirements: Does not return
Registers affected: Does not return

**Description:** This routine switches from BASIC to machine language monitor mode. It does not return to the caller. When the user quits the monitor, it will restart BASIC.

**How to Use:**
1) Call this routine.

**EXAMPLE:**

      JMP MONITOR

## Machine Language Monitor

The built-in machine language monitor can be started with the `MON` BASIC command. It is based on the monitor of the Final Cartridge III and supports all its features. See the [Final Cartridge III Manual](https://rr.pokefinder.org/rrwiki/images/7/70/Final_Cartridge_III_english_Manual.pdf) more more information.

Some features specific to this monitor are:
* The `I` command prints a CBM-ASCII-encoded memory dump.
* The `EC` command prints a binary memory dump. This is also useful for character sets.
* Scrolling the screen with the cursors or F3/F5 will continue memory dumps and disassemblies, and even disassemble backwards.

The following additions have been made:

* The instruction set extensions of the 65C02 are supported.
* The `O` command takes an 8 bit hex value as an argument and sets it as the ROM and RAM bank for reading and writing memory contents. The following example disassembles the beginning of the DOS ROM on bank 2:

      O02
      DC000 C015

* The `OV` command takes a 4 bit hex value as an argument and sets it as the bank in the video address space for reading and writing memory contents. The following example shows the character ROM in the video controller's address space:

      OV2
      EC0000 000F

*[TODO: Full documentation]*

## Memory Map

The Commander X16 has 64 KB of ROM and 2,088 KB (2 MB[^1] + 40 KB) of RAM. Some of the ROM and RAM is always visible at certain address ranges, while the remaining ROM and RAM is banked into one of two address windows.

This is an overview of the X16 memory map:

|Addresses  |Description                                                       |
|-----------|------------------------------------------------------------------|
|$0000-$9EFF|Fixed RAM (40 KB minus 256 bytes)								   |
|$9F00-$9FFF|I/O Area (256 bytes)											   |
|$A000-$BFFF|Banked RAM (8 KB window into one of 256 banks for a total of 2 MB)|
|$C000-$DFFF|Banked ROM (8 KB window into one of 8 banks for a total of 64 KB) |
|$E000-$FFFF|Fixed ROM (KERNAL)												   |

### Banked Memory

The RAM bank (0-255) defaults to 255, and the ROM bank (0-7) defaults to 7 on RESET. The RAM bank can be configured through VIA#1 PA0-7 ($9F61), and the ROM bank through VIA#1 PB0-2 ($9F60). The section  "I/O Programming" for more information.

### ROM Allocations

The fixed ROM at $E000-$FFFF contains the KERNAL. This is the allocation of the banks of banked ROM:

|Bank|Name |Description                                            |
|----|-----|-------------------------------------------------------|
|0   |BASIC|The BASIC interpreter                                  |
|1   |UTIL |Utilities like the machine language monitor            |
|2   |DOS  |The computer-based CBM-DOS for FAT32 SD cards          |
|3-7 |     |[Unassigned]                                           |

### RAM Contents

This is the allocation of fixed RAM in the KERNAL/BASIC environment.

|Addresses  |Description                                                     |
|-----------|----------------------------------------------------------------|
|$0000-$00FF|KERNAL and BASIC zero page variables                            |
|$0100-$01FF|CPU stack                                                       |
|$0000-$03FF|KERNAL and BASIC variables                                      |
|$0800-$9EFF|BASIC program/variables; available to the user                  |

The following zero page locations are unused by KERNAL/BASIC and are available to the user:

|Addresses  |
|-----------|
|$00FB-$00FF|

In a machine language application that only uses KERNAL, the following zero page locations are also available:

|Addresses  |
|-----------|
|$0003-$008F|

This is the allocation of banked RAM in the KERNAL/BASIC environment.

|Bank   |Description               |
|-------|--------------------------|
|0-254  |Available to the user     |
|255[^2]|DOS buffers and variables |

### I/O Area

This is the memory map of the I/O Area:

|Addresses  |Description                  |
|-----------|-----------------------------|
|$9F00-$9F1F|Reserved for audio controller|
|$9F20-$9F3F|VERA video controller		  |
|$9F40-$9F5F|Reserved					  |
|$9F60-$9F6F|VIA I/O controller #1		  |
|$9F70-$9F7F|VIA I/O controller #2		  |
|$9F80-$9F9F|Real time clock			  |
|$9FA0-$9FBF|Future Expansion			  |
|$9FC0-$9FDF|Future Expansion			  |
|$9FE0-$9FFF|Future Expansion			  |

## Video Programming

The VERA video chip supports resolutions up to 640x480 with up to 256 colors from a palette of 4096, two layers of either a bitmap or tiles, 128 sprites of up to 64x64 pixels in size. It can output VGA as well as a 525 line interlaced signal, either as NTSC or as RGB (Amiga-style).

See [vera-module v0.6.pdf] for the complete reference.

## Sound Programming

*[TODO]*

## I/O Programming

There are two 65C22 "Versatile Interface Adapter" (VIA) I/O controllers in the system, VIA#1 at address $9F60 and VIA#2 at address $9F70. The IRQ out lines of both VIAs are connected to the IRQ in line of the CPU.

The following tables describe the connections of the GPIO ports:

**VIA#1**

|Pin  |Description |
|-----|------------|
|PA0-7|RAM bank    |
|PB0-2|ROM bank    |
|PB3-7|*[TBD]*       |

**VIA#2**

|Pin  |Description     |
|-----|------------    |
|PA0  |PS/2 DAT        |
|PA1  |PS/2 CLK        |
|PA2  |TBD             |
|PA3  |JOY1/2 LATCH[^3]|
|PA4  |JOY1 DATA       |
|PA5  |JOY1/2 CLK      |
|PA6  |JOY2 DATA       |
|PA7  |*[TBD]*         |
|PB0-7|*[TBD]*         |

<!------->

[^1]: Current development systems have 2 MB of bankable RAM. Actual hardware is currently planned to have an option of either 512 KB or 2 MB of RAM.

[^2]: On systems with 512 KB RAM, DOS uses bank 63, and banks 0-62 are available to the user.

[^3]: The pin assignment of the NES/SNES controller is likely to change.