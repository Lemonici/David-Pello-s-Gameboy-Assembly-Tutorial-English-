# Gameboy Assembly Tutorial

---

#### Translator's note
An improved English translation of the GBZ80 tutorial found [here](http://wiki.ladecadence.net/doku.php?id=tutorial_de_ensamblador) and the original translation [here](https://gb-archive.github.io/salvage/tutorial_de_ensamblador/tutorial_de_ensamblador%20%5BLa%20decadence%5D.html)


While trying to learn assembly for the Gameboy I came accross this tutorial. Unfortunately, the translation was abysmal, making it pretty hard to follow. I wanted to contribute to the community and my Spanish skills are considerably better than my programming knowledge (Advanced-Mid on the ACTFL, mom is very proud). I figured it may be of use to provide a better translation for those of you who don't consider 2 years of foreign language experience a reasonable requirement for learning an unrelated programming language.

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
### Nintendo Gameboy (DMG)

To start this Gameboy development tutorial, we'll briefly review the technical specifications of our wonderful console:

+ CPU: 8-bit SHARP LR35902 (similar to the Z80) running at 4.19 MHz
+ 8KiB Internal RAM
+ 8KiB Video RAM
+ 2.6" LCD display, 160 x 144
+ Stereo sound, 4 channels
+ 6V, 0.7A
+ 90mm x 148mm x 32mm

Alright, so in the heart of the Gameboy we have a CPU manufactured by Sharp specifically for Nintendo, we have a microprocessor halfway between the 8080 and the Z80, since even though it doesn't have the extra sets of registers or the indexes of the Z80, it does have most of it's extended instruction set, like those used in bit manipulation. It also includes additional circuitry to control the display, the joypad, the serial port, and for audio generation.

This tutorial doesn't claim to be a complete programming tutorial for the Z80 (or, in this case, the GBz80 as it's often called), but instead focuses on the Gameboy hardware and how to use it. To learn Z80 assembly I recommend existing documentation like the complete instruction set for the Gameboy CPU at http://gbdev.gg8.se/wiki/articles/CPU_Instruction_Set and the Z80 assembly course for the Spectrum at https://wiki.speccy.org/cursos/ensamblador/indice [This link is in Spanish]. Remember that the Z80 has a few instructions that the Gameboy CPU doesn't have, but as for the rest of them learning Z80 assembly will be perfect for the Gameboy (as well as the Spectrum, Amstrad CPC, MSX, Sega Master System...).

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
The Gameboy's primary memory is mapped in a 16-bit space and allows us to directly address 64KiB (2^16 = 65536) locations. In this address space we need to assign addresses to all the memory blocks that the Gameboy needs to access. This includes the RAM, the cartridge ROM, the cartridge's internal RAM for games that save, video memory, etc. To this end, the Gameboy's designers mapped the memory to different necessary blocks like internal RAM or video memory, leaving two 16KiB blocks for accessing a game's ROM, and 8KiB to access a game's RAM (savegames). Since a lot of games needed more than 32KiB of ROM or 8KiB of save RAM, programmers started using a technique called 'Banking' in which the game's ROM is divided in different blocks that can be made independent (the graphics or sounds of different screens, for example), that are mapped in the memory access block as necessary. In the Gameboy, this was designed as follows; we have one static 16KiB block (where we program the main game logic) and then, through specific instructions (depending on the mapping chip we use in our cartridge) we can swap out different 16KiB banks in the open block. It seems complicated, but we'll talk about banking later on. This is all seen in the following memory map with all the available blocks in the address space of the Gameboy.
```
 General memory map*                       Bank writing registers [I'm not sure about this one
 									I had a hard time finding an English equivalent]
 -------------------                       ----------------------

  Interrupt activation registers
 ---------------------------------- $FFFF
  High internal RAM (HRAM)
 ---------------------------------- $FF80
  Unusable
 ---------------------------------- $FF4C
  I/O ports
 ---------------------------------- $FF00
  Unusable
 ---------------------------------- $FEA0
  Sprite attributes (OAM)
 ---------------------------------- $FE00
  Echo RAM 
 ---------------------------------- $E000
  8kB internal RAM (WRAM)
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
