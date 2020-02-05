# Gameboy Assembly Tutorial

---

#### Translator's note
An improved English translation of the GBZ80 tutorial found here: http://wiki.ladecadence.net/doku.php?id=tutorial_de_ensamblador.

While trying to learn assembly for the Gameboy I came accross this tutorial. Unfortunately, the translation was abysmal, making it pretty hard to follow. I wanted to contribute to the community and my Spanish skills are considerably better than my programming knowledge (Advanced-Mid on the ACTFL, mom is very proud). I figured it may be of use to provide a better translation for those of you who don't consider 2 years of foreign language experience a reasonable requirement for learning an unrelated programming language.

As a result of my Spanish skills being better than my programming skills,

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
+ 2.6" LCD display, 160 \times 144
+ Stereo sound, 4 channels
+ 6V, 0.7A
+ 90mm \times 148mm \times 32mm

Alright, so in the heart of the Gameboy we have a CPU manufactured by Sharp specifically for Nintendo, we have a microprocessor halfway between the 8080 and the Z80, since even though it doesn't have the extra sets of registers or the indexes of the Z80, it does have most of it's extended instruction set, like those used in bit manipulation. It also includes additional circuitry to control the display, the joypad, the serial port, and for audio generation.

This tutorial doesn't claim to be a complete programming tutorial for the Z80 (or, in this case, the GBZ80 as it's often called), but instead focuses on the Gameboy hardware and how to use it. To learn Z80 assembly I recommend existing documentation like the complete instruction set for the Gameboy CPU at http://gbdev.gg8.se/wiki/articles/CPU_Instruction_Set and spectrum's Z80 assembly course at https://wiki.speccy.org/cursos/ensamblador/indice [Translator's].
```

```
  
  
