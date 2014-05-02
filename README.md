ECE281_CE5
==========

MIPS 

### Task 1: MIPS Program

The first task of the MIPS lab is to create a MIPS assembly code program that will load a decimal 44 and -37 to registers $s0 and $S1 respectively, add them into register $S2, then store that value to hexadecimal memory location 0x54. The four line program is given below:

    1. addi $S0, $0,  44    # $s0 = 44+0
    2. addi $S1, $0,  -37   # $s1 = -37+0 
    3. add  $s2, $s0, $s1   # $s2 = $s0 + $s1 
    4. sw   $s2, 0x54($0)   # mem(0x54) <= $s2
    
### Task 2: Machine code realization

The program above is translated into hexadecimal machine code below each line is translated in regards to the instruction type (for this program only I-type or R-type instructions).

#### I-Type Instructions

The I-Type instructions in the program are shown below with the instruction machine code structure:

    1. addi $S0, $0,  44 
    2. addi $S1, $0,  -37
    4. sw   $s2, 0x54($0)
    
|Opcode|rs|rt|Imm|
|6 bits|5 bits|5 bits|16 bits|

Line 1:

|Label|value|Decimal|Binary|
|opcode|addi|8|001000|
|rt|$s0|16|10000|
|rs|$0|0|00000|
|imm|44|--|0000 0000 0010 1100|

Concatenated binary:

    0010 0000 0001 0000 0000 0000 0010 1100

Hexadecimal:

    0x2010002C

Line 2:

|Label|value|Decimal|Binary|
|opcode|addi|8|001000|
|rt|$s1|17|10001|
|rs|$0|0|00000|
|imm|-37|--|1111 1111 1101 1011|

Concatenated binary:

    0010 0000 0001 0001 1111 1111 1101 1011
    
Hexadecimal:

    0x2011FFDB
    
Line 4: 

|Label|value|Decimal|Binary|
|opcode|sw|43|101011|
|rt|$s2|18|10010|
|rs|$0|0|00000|
|imm|0x54|--|0000 0000 0010 1100|

Concatenated binary:

    1010 1100 0001 0010 0000 0000 0010 1100
    
Hexadecimal:

    0xAC12002C
    
#### R-type Instructions

The single R-type instruction is:

    3. add  $s2, $s0, $s1   # $s2 = $s0 + $s1 
    
and the machine code implementation

|Opcode|rs|rt|rd|shamt|func|
|6 bits|5 bits|5 bits|5 bits|6bits|

The Binary translation of line 3 is then:

|Label|value|Decimal|Binary|
|opcode|R-Type|0|000000|
|rd|$s2|18|10010|
|rt|$s1|11|10001|
|rs|$s0|16|10000|
|func|add|32|100000|

Binary concatenation

    0000 0010 0001 0001 1001 0000 0010 0000
    
Hexadecimal:

    0x02119020
    
#### Result

The hexadecimal machine code for the program above is therefor

|0x2010002C|
|0x2011FFDB|
|0x02119020|
|0xAC12002C|
