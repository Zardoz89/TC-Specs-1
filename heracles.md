Heracles Text Adapter
----


|     Item       |   Value    |   Comment
| -------------: | ---------- | ----------------
|    Vendor code | 0xBEBECAFE | Investronics Research Ltd.
|      Device ID | 0x734D0084 | Heracles Text Adapter
|    Device type | 0x734D     | Memory mapped text display
|        Version | 0x0084     |

The Heracles Text Adapter, offers an powerful text display device that it's backwards compatible with the popular Nya Elektriska LEM1802.
Plus the LEM1802 compatibility mode, have this features:

- 32x12 color Text mode with 4x8 font (efective 128x96 pixels)
- 32x12 color Text mode with 8x8 font (efective 256x96 pixels)
- 80x25 color Text mode with 4x8 font (efective 320x200 pixels)
- 40x25 color Text mode with 8x8 font (efective 320x200 pixels)
- 256 Glyphs fonts
- Toggle blink attribute
- Hardware controlled cursor

Interrupt Functionality
----
When the device is interrupted, the A register will be interpreted as a command

- **0x0000**: Map Text buffer

  Register B should be set to the base memory address of the text buffer ram.
    - Requires from 384 to 2000 words depending of the actual text mode.
    - If B is 0, the display is disabled.
       
- **0x0001**: Map Font RAM

  Register B should be set to the base memory address of the font ram.
    - Requires from 256 to 1024 words depending of the size of the font and if is 256 glyphs fonts are enabled
    - If B is 0, the ROM fonts are used instead. (Default behavior)

- **0x0002**: Map Palette

  Register B should be set to the base memory address of the palette ram
    - Requires 32 words
    - If B is 0, the default palette is used instead.
    
- **0x0003**: Border Color

  Register B should be set to the palette color index. Only the lower 8 bites are read.
  Only works on LEM1802 compatible modes (ie 32x16 with 4x8 fonts)

- **0x0004**: Report Capabilities

  Register B should be set to an address where begin writing display mode descriptors.
	Register C should be set to maximum number of descriptors to write. If the program does not provide enough memory to record all modes the display is capable of, some modes may not be detected. Setting C to 0, will write all modes

	Up to 3 * C words will be written to memory begining at B.
	
- **0x0005**: Set display mode

	CXY should contain the complete display mode descriptor to be set.
	B will be set to an error code. Note that 0x0000 is never used.
    - 0x0001: Mode set.
    - 0x0002: Mode not supported.
	
- 0x0006 through 0x001F: Reserved for future expansion.

- **0x0020**: Configure behavior
  
  Register B should set acording a mask of bits to enable/disable this features:
    - 128 or 256 glyphs fonts
    - Enable/disable blinking (4 bit of background color)

- **0x0021**: Sets Harware cursor
  
  Register B should set to the mask values indicated on Harware Cursor section.
    - If B is 0, disables harware cursor.


Video ram
----
  The behavour it's the same that LEM1802. But with a few differences.
    
  ### Size
    
  The size of the video ram depends of the actual video mode :
    - 32x12 : 384 words
    - 40x25 : 1000 words
    - 80x25 : 2000 words
      
  ### 256 glyphs fonts and blink bit
    
  If the 256 glyphs fonts are enabled, the format of a word changes (LSB format ):
    
```
    f f f f B b b b c c c c c c c c
```
  Where :
    - ffff defines what color use as foreground (ink) from the palette
    - Bbbb defines what color use as background from the palette
    - cccccccc defines what glyph of the font is displayed
      
  Also, if blinking is enabled, then :
    - bbb defines what color use as background from the palette. Only allows to use the first 8 colors of the pallete
    - If B is set, the characted color will blink.
    
TODO:    

Font ram
----

  The LEM1802 has a default built in font. If the user chooses, they may
  supply their own font by mapping a 256 word memory region with two words
  per character in the 128 character font.
  By setting bits in these words, different characters and graphics can be
  achieved. For example, the character F looks like this:
       word0 = 1111111100001001
       word1 = 0000100100000000
  Or, split into octets:
       word0 = 11111111 /
               00001001
       word1 = 00001001 /
               00000000
    

Palette ram
----

  The LEM1802 has a default built in palette. If the user chooses, they may
  supply their own palette by mapping a 16 word memory region with one word
  per palette entry in the 16 color palette.
  Each color entry has the following bit format (in LSB-0):
       0000rrrrggggbbbb
  Where r, g, b are the red, green and blue channels. A higher value means a
  lighter color.
  
Hardware cursor
----
