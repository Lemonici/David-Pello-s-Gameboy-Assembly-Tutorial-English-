# Game Boy Assembly Tutorial

---

#### Translator's note
An improved English translation of David Pello's GBZ80 tutorial found [here](http://wiki.ladecadence.net/doku.php?id=tutorial_de_ensamblador) and the original translation [here](https://gb-archive.github.io/salvage/tutorial_de_ensamblador/tutorial_de_ensamblador%20%5BLa%20decadence%5D.html)


While trying to learn assembly for the Game Boy I came accross this tutorial. Unfortunately, the translation was abysmal, making it pretty hard to follow. I wanted to contribute to the community and my Spanish skills are considerably better than my programming knowledge (Advanced-Mid on the ACTFL, mom is very proud). I figured it may be of use to provide a better translation for those of you who don't consider 2 years of foreign language experience a reasonable requirement for learning an unrelated programming language.

As a result of my Spanish skills being better than my programming skills, I was unable to preserve the syntax hilighting found in the original document. I also don't have a perfect grasp of the technical vocabulary in English, so please let me know if there are any errors.

Any future comments not in the original document will be in [square brackets].

---

# Introduction
```
    NINTENDO
   ______                     ____             
  / ____/___ _____ ___  ___  / __ )____  __  __
 / / __/ __ `/ __ `__ \/ _ \/ __  / __ \/ / / /
/ /_/ / /_/ / / / / / /  __/ /_/ / /_/ / /_/ / 
\____/\__,_/_/ /_/ /_/\___/_____/\____/\__, /  
                                      /____/   
			Programming Tutorial
			 By David Pello (ladecadence.net)
			 Translated by Patrick Pollock (Lemonici)
             _n_________________     
            |_|_______________|_|    
            |  ,-------------.  |    
            | |  .---------.  | |    
            | |  |         |  | |    
            | |  |         |  | |    
            | |  |         |  | |    
            | |  |         |  | |    
            | |  `---------'  | |   
            | `---------------' |  
            |   _ GAME BOY      |    
            | _| |_         ,-. |    
            ||_ O _|   ,-. "._,"|    
            |  |_|    "._,"   A | 
            |      _  _  B      |
            |     // //         |    
            |    // //  \\\\\\  |    
            |    `  `    \\\\\\ ,    
            |________...______,"     

```
### Nintendo Game Boy (DMG)

To start this Game Boy development tutorial, we'll briefly review the technical specifications of our wonderful console:

+ CPU: 8-bit SHARP LR35902 (similar to the Z80) running at 4.19 MHz
+ 8KiB Internal RAM
+ 8KiB Video RAM
+ 2.6" LCD display, 160 x 144
+ Stereo sound, 4 channels
+ 6V, 0.7A
+ 90mm x 148mm x 32mm

Alright, so in the heart of the Game Boy we have a CPU manufactured by Sharp specifically for Nintendo, we have a microprocessor halfway between the 8080 and the Z80, since even though it doesn't have the extra sets of registers or the indexes of the Z80, it does have most of it's extended instruction set, like those used in bit manipulation. It also includes additional circuitry to control the display, the joypad, the serial port, and for audio generation.

This tutorial doesn't claim to be a complete programming tutorial for the Z80 (or, in this case, the GBz80 as it's often called), but instead focuses on the Game Boy hardware and how to use it. To learn Z80 assembly I recommend existing documentation like the complete instruction set for the Game Boy CPU at http://gbdev.gg8.se/wiki/articles/CPU_Instruction_Set and the Z80 assembly course for the Spectrum at https://wiki.speccy.org/cursos/ensamblador/indice [This link is in Spanish]. Remember that the Z80 has a few instructions that the Game Boy CPU doesn't have, but as for the rest of them learning Z80 assembly will be perfect for the Game Boy (as well as the Spectrum, Amstrad CPC, MSX, Sega Master System...).

### GBz80
The CPU, which we will call the GBz80, is an 8-bit CPU with a 16-bit address bus. In other words, the internal data and extermal memory are organized in bytes and can address 2^16 = 64KiB of external memory.

#### Registers
The GBz80 has several internal registers where we can store data while we manipulate and move them to and from the external memory to the CPU. These 8-bit registers are `a`, `b`, `c`, `d`, `e`, `h`, and `l`.

#### Flags
There's also a special `f` register that saves the state of the processor flags. As the processor performs certain operations it can set or reset some of these flags, which will be very useful to us as we program. For example, one of the bits in this register is the Zero flag, which tells us if the result of the previous operation was a zero or not. Not all of the operations modify the flags, but many do. To understandin depth all the operations that the GBz80 can perform, you can take a look at the related section in the Pan Docs: http://gbdev.gg8.se/wiki/articles/CPU_Instruction_Set.

The `f` register's flags are as follows:
```
Bit  Name (1) (0)   Meaning
7    z     Z   NZ   Zero 
6    n     -   -    Add/Subtract (BCD)
5    h     -   -    Half-carry (BCD)
4    c     C   NC   Carry
3-0  -     -   -    Unused (always set to 0)
```

The most commonly used flags are:
##### The Zero flag `Z`
This bit is set to 1 if an operation's result is 0. This is widely used in conditional jumps.
##### The Carry flag `C`
This bit is set to 1 when a summation's result is greater than $FF (8-bit) or $FFFF (16-bit), or when the result of a subtraction or comparison is less than 0. It's also set to 1 when a rotate or shift instruction pulls out a 1. It's used in conditional jumps or for instructions like `ADC`, `SBC`, `RL`, `RLA`, etc.
##### 16-bit Registers
Some of these registers can be combined to form 16-bit registers. They're very useful for managing memory addresses or larger numbers if we need them. The 16-bit registers are `af`, `bc`, `de`, `hl`.

#### Memory Map
The Game Boy's primary memory is mapped in a 16-bit space and allows us to directly address 64KiB (2^16 = 65536) locations. In this address space we need to assign addresses to all the memory blocks that the Game Boy needs to access. This includes the RAM, the cartridge ROM, the cartridge's internal RAM for games that save, video memory, etc. To this end, the Game Boy's designers mapped the memory to different necessary blocks like internal RAM or video memory, leaving two 16KiB blocks for accessing a game's ROM, and 8KiB to access a game's RAM (savegames). Since a lot of games needed more than 32KiB of ROM or 8KiB of save RAM, programmers started using a technique called 'Banking' in which the game's ROM is divided in different blocks that can be made independent (the graphics or sounds of different screens, for example), that are mapped in the memory access block as necessary. In the Game Boy, this was designed as follows; we have one static 16KiB block (where we program the main game logic) and then, through specific instructions (depending on the mapping chip we use in our cartridge) we can swap out different 16KiB banks in the open block. It seems complicated, but we'll talk about banking later on. This is all seen in the following memory map with all the available blocks in the address space of the Game Boy.
```
 General memory map*                       Bank writing registers
 -------------------                       ----------------------

  Interrupts Enable Register (IE)
 ---------------------------------- $FFFF
  High RAM (HRAM)
 ---------------------------------- $FF80
  Unusable
 ---------------------------------- $FF4C
  I/O Registers
 ---------------------------------- $FF00
  Unusable
 ---------------------------------- $FEA0
  Sprite attribute table (OAM)
 ---------------------------------- $FE00
  Mirror of C000-DDFF (ECHO RAM)
 ---------------------------------- $E000
  8kB Work RAM (WRAM)
 ---------------------------------- $C000       ------------------------------
  8kB swappable RAM bank (SRAM)                /      MBC1 ROM/RAM selector
 ---------------------------------- $A000     /  -----------------------------
  8kB Video RAM                              /  /     RAM bank selector
 ---------------------------------- $8000 --/  /  ----------------------------
  16kB swappable ROM bank           $6000 ----/  /    ROM bank selector
 ---------------------------------- $4000 ------/  ---------------------------
  16kB ROM bank 0                   $2000 --------/   Bank Activation RAM
 ---------------------------------- $0000 ------------------------------------
```
\* **NOTE:** b = bit, B = byte

\* **NOTE:** Following the conventions of RGBDS, the assembler we're using in this tutorial, I'm writing the numbers in hexadecimal notation with a leading '$', the ones in binary with a leading '%', and decimals without a prefix

#### Game Boy Rom Organization
Game Boy ROMs need to have a certain structure for the Game Boy to accept them, especially when talking about the header. Below I describe in detail the parts of a ROM and their functions:
```
GAMEBOY IMAGE DIAGRAM
---------------------

$0    -  $100:	Interrupt vectors, but these addresses can also be used for
		inserting our own code
				
$100  -  $103:	Entry point for executing the program, it's common practice to put
		a 'nop' follow by a jump to our entry point.
				
$104  - $14E:	Cartridge header. Contains the Nintendo logo ($104-$133)
		which is compared with the existing one when the console boots.
		If they aren't the same, the program stops executing
		
		After that we have the following data:
		
		$134 - Cartridge Name - 15 bytes
		
		$143 - Game Boy Color Support
			$80 = Game Boy Color, $00 or otherwise = B/W
					
		$144 - Licensee Code, 2 bytes (unimportant)
		
		$146 - Super Game Boy Support (SGB)
			$00 = Game Boy, $03 = Super Game Boy
					
		$147 - Cartridge Type (ROM only, MBC1, MBC1+RAM.. etc)
			 0 - ROM ONLY                12 - ROM+MBC3+RAM
			 1 - ROM+MBC1                13 - ROM+MBC3+RAM+BATT
			 2 - ROM+MBC1+RAM            19 - ROM+MBC5
			 3 - ROM+MBC1+RAM+BATT       1A - ROM+MBC5+RAM
			 5 - ROM+MBC                 1B - ROM+MBC5+RAM+BATT
			 6 - ROM+MBC2+BATT           1C - ROM+MBC5+RUMBLE
			 8 - ROM+RAM                 1D - ROM+MBC5+RUMBLE+SRAM
			 9 - ROM+RAM+BATT            1E - ROM+MBC5+RUMBLE+SRAM+BATT
			 B - ROM+MMM01               1F - GBCamera
			 C - ROM+MMM01+SRAM          FD - Bandai TAMA5
			 D - ROM+MMM01+SRAM+BATT     FE - Hudson HuC-3
		 	 F - ROM+MBC3+TIMER+BATT     FF - Hudson HuC-1
			10 - ROM+MBC3+TIMER+RAM+BATT
			11 - ROM+MBC3
			
		$148 - ROM Size 
			  0 - 256Kbit =  32KByte =   2 bancos
		  	  1 - 512Kbit =  64KByte =   4 bancos
		   	  2 -   1Mbit = 128KByte =   8 bancos
		  	  3 -   2Mbit = 256KByte =  16 bancos
		  	  4 -   4Mbit = 512KByte =  32 bancos
		  	  5 -   8Mbit =   1MByte =  64 bancos
		  	  6 -  16Mbit =   2MByte = 128 bancos
                          7 -  32Mbit =   4MByte = 256 bancos
			$52 -   9Mbit = 1.1MByte =  72 bancos
			$53 -  10Mbit = 1.2MByte =  80 bancos
			$54 -  12Mbit = 1.5MByte =  96 bancos
			
		$149 - RAM Size
			0 - None
			1 -  16kBit =   2kB =  1 bank
			2 -  64kBit =   8kB =  1 bank
			3 - 256kBit =  32kB =  4 banks
			4 -   1MBit = 128kB = 16 banks
			
		$14A - Destination Code
			0 - Japanese
			1 - Non-Japanese
			
		$14B - Old Licensee Code, 2 bytes 
			$33 - Look for code in $0144/$0145
			(SGB functions don't work unless this is set to $33)
		
		$14C - Mask ROM Version (Normally $00)
		
		$14D - Header Checksum (important)
		
		$14E - Golbal Checksum (not important)

$14F  - $3FFF:	Our code. This is the 16KiB bank 0, and is fixed.

$4000 - $7FFF:	The second 16KiB bank. When the Game Boy boots this
		corresponds with bank 1 (And goes to 32KiB of the ROM), but it
		can be swapped for banks 2+ (each with 16KiB) up to the last
		available bank using a mapper like the MBC1
```

#### Video System
The Game Boy's video system doesn't access individual pixels like a PC does. Instead, it's a system of blocks or "tiles". It consists of 256 x 256 pixel buffer (32 x 32 tiles) of which 160 x 144 can be shown at a given time (making the screen size 20 x 18 tiles). We have a pair of registers, `SCROLLX` and `SCROLLY` that allow us to move the visible area around the buffer. The buffer is also 'circular'; when the visible area scrolls past the edge of the buffer it wraps around and shows data from the opposite side.

```
				  ^
				  |
				  v

		+------------------------------------+
		|(scrollx, scrolly)                  |
		|   +----------------+               |
		|   |                |               |
		|   |                |               |
		|   |                |               |
		|   |     20x18      |               |
		|   |  visible area  |               |
		|   |                |               |
		|   |                |               |
	<->	|   |                |               |	<->
		|   +----------------+               |
		|                                    |
		|                32x32               |
		|           background map           |
		|                                    |
		|                                    |
		|                                    |
		|                                    |
		|                                    |
		+------------------------------------+		
				  ^
				  |
				  v
```

The registers that control the video system are very important since they allow us to control what and how the screen displays. The most important ones are:

##### $FF40 - LCDC - LCD Control (R/W)
This control register allows us to adjust the Game Boy's screen and should be used to manage any operation that involves displaying something with it.

Every bit in this register has a special significance, as explained below
```
 Bit 7 - LCD Display Enable             (0=Off, 1=On)
 Bit 6 - Window Tile Map Display Select (0=9800-9BFF, 1=9C00-9FFF)
 Bit 5 - Window Display Enable          (0=Off, 1=On)
 Bit 4 - BG & Window Tile Data Select   (0=8800-97FF, 1=8000-8FFF)
 Bit 3 - BG Tile Map Display Select     (0=9800-9BFF, 1=9C00-9FFF)
 Bit 2 - OBJ (Sprite) Size              (0=8x8, 1=8x16)
 Bit 1 - OBJ (Sprite) Display Enable    (0=Off, 1=On)
 Bit 0 - BG/Window Display/Priority     (0=Off, 1=On)
```

Now we'll explain the functions of some of the bits because they're so important:

###### Bit 7 - Display Control
Activates and deactivates the LCD. The display has to be activated before we can use it to display something, and sometimes we want to deactivate it if we need to write a lot of data to the screen, like a presentation image or end screen, so that there's time to put all the graphics in before the screen starts to draw them and we see half-completed graphics and other oddities.

###### ATTENTION
The display can only be activated and deactivated when we're in the vertical interval period (VBlank). Activating or deactivating it outside of this period can damage the hardware. Be extremely careful with this since nothing will happen in an emulator but you can ruin your Game Boy if you do it with real hardware. So to activate or deactivate the LCD, always wait for the vertical interval (we'll see how to do that later).

###### Bit 0 - Background Control
If this bit is 0 the background isn't drawn, it stays blank (so to draw the background we have to set this bit to 1, obviously). We've seen this in many games for flickering effects and the like.

The rest of the bits act similarly and I think they're adequately explained in the table.

##### $FF41 - Status - LCD (R/W) Status
This register is also very important, since it allows us to know what the LCD is doing in a given moment. We need to check it for a number of operations, so pay attention. It also allows us to activate the LCD interrutps so, as you can see, we have a lot of functionality in this register.

An explanatory table and then we'll go over it in more detail:
```
 Bit 6 - LYC=LY Coincidence Interrupt (1=Enable) (Read/Write)
 Bit 5 - Mode 2 OAM Interrupt         (1=Enable) (Read/Write)
 Bit 4 - Mode 1 V-Blank Interrupt     (1=Enable) (Read/Write)
 Bit 3 - Mode 0 H-Blank Interrupt     (1=Enable) (Read/Write)
 Bit 2 - Coincidence Flag  (0:LYC<>LY, 1:LYC=LY) (Read Only)
 Bit 1-0 - Mode Flag       (Mode 0-3, see below) (Read Only)
           0: During H-Blank
           1: During V-Blank
           2: During Searching OAM
           3: During Transferring Data to LCD Driver
```
