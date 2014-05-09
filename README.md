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
|:-:|:-:|:-:|:-:|
|6 bits|5 bits|5 bits|16 bits|

Line 1:

|Label|Value|Decimal|Binary|
|:-:|:-:|:-:|:-:|
|opcode|addi|8|001000|
|rt|$s0|16|10000|
|rs|$0|0|00000|
|imm|44|--|0000 0000 0010 1100|

Concatenated binary:

    0010 0000 0001 0000 0000 0000 0010 1100

Hexadecimal:

    0x2010002C

Line 2:

|Label|Value|Decimal|Binary|
|:-:|:-:|:-:|:-:|
|opcode|addi|8|001000|
|rt|$s1|17|10001|
|rs|$0|0|00000|
|imm|-37|--|1111 1111 1101 1011|

Concatenated binary:

    0010 0000 0001 0001 1111 1111 1101 1011
    
Hexadecimal:

    0x2011FFDB
    
Line 4: 

|Label|Value|Decimal|Binary|
|:-:|:-:|:-:|:-:|
|opcode|sw|43|101011|
|rt|$s2|18|10010|
|rs|$0|0|00000|
|imm|0x54|--|0000 0000 0101 0100|

Concatenated binary:

    1010 1100 0001 0010 0000 0000 0101 0100
    
Hexadecimal:

    0xAC120054
    
#### R-type Instructions

The single R-type instruction is:

    3. add  $s2, $s0, $s1   # $s2 = $s0 + $s1 
    
and the machine code implementation

|Opcode|rs|rt|rd|shamt|func|
|:-:|:-:|:-:|:-:|:-:|:-:|
|6 bits|5 bits|5 bits|5 bits|6bits|

The Binary translation of line 3 is then:

|Label|Value|Decimal|Binary|
|:-:|:-:|:-:|:-:|
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

    1. 0x2010002C
    2. 0x2011FFDB
    3. 0x02119020
    4. 0xAC120054

#### Testing

The MIPS program was tested in the MIPS VHDL simulator provided. The instructions were loaded individually by way of a testbench program and the output read with the waveform shown below. 

![alt text](https://raw.githubusercontent.com/IanGoodbody/ECE281_CE5/master/waveform.jpg "Output waveform")

The three registers of interest, $s0 mem(16), $s1 mem(17), and $s2 mem(18), are highlighted in yellow. The instructions are shown in the expected order in the "instr" bus. Because this is a single cycle MIPS machine the values update within the same instruction period and the instructions all take the same amount of time. The program is shown to work in the waveform as all the register values are updated with the proper value during the proper instruction. The "sw" command can be verified by noting that the "memwrite" signal is active, the "aluout" bus is sending the address value 0x54 to memory, and the "writedata" bus is sending the value 0x7 from $s2 into the memory.

### Task 3 programing 

The final task was to implement the ORI or "or immediate" functionality to the MIPS  program. Because the MIPS datapath is already configured to implement immediate instructions like ANDI, the datapath will not need to be modified only the constructor. Additionally, because the controller implements logical immediate instructions like ANDI the only signal that would need to be changed from such an instruction would be the three bit "alucontrol" bus which requires a "001" for the or instruction. As for the specifications of the ORI instruction, appendix B of the course text indicates that the instruction has a function code of 13 or 001101 and has the same input format as other I-type instructions shown above.

Implementing the ORI functionality required adding a new state to the controller for the opcode "001101" the two modifications are shown below:

Lines 188 - 199

    process(op) begin
        case op is
          when "000000" => controls <= "110000010"; -- Rtype
          when "100011" => controls <= "101001000"; -- LW
          when "101011" => controls <= "001010000"; -- SW
          when "000100" => controls <= "000100001"; -- BEQ
          when "001000" => controls <= "101000000"; -- ADDI
    	  when "001101" => controls <= "101000010"; -- ORI --ADDED
          when "000010" => controls <= "000000100"; -- J
          when others   => controls <= "---------"; -- illegal op
        end case;
    end process;
    
Lines 215 - 217

    when "00" => alucontrol <= "010"; -- add (for lb/sb/addi)
    when "01" => alucontrol <= "110"; -- sub (for beq)
    when "10" => alucontrol <= "001"; -- or  (for ori) --ADDED
    
The changes to the controller included adding a new case for the ORI opcode, then adding another "aluop" signal "10" to push cause an or operation.

The ORI in instruction was tested with the following assembly code which was added to the previous testbench:

    SUB $s0, $s0, $s0, # $s0 = 0 (resets $s0 to 0)
    ADDI $s0, $0, 0xABCD # $s0 = 0 + 0xffffabcd 
    ORI $s1, $s0, 0xFFFF # $s1 = $s0 | 0xffffffff = 0xffffffff
    ORI $s2, $s0, 0xABCD # $s1 = $s0 | 0xffffabcd = 0xffffabcd
    ORI $s3, $s0, 0x5432 # $s1 = $s0 | 0x00005432 = 0xffffffff
    
With the machine code reading

    1. 0x02108022
    2. 0x2010ABCD
    3. 0X3611FFFF
    4. 0x3612ABCD
    5. 0x36135432
    
The testbench waveform is shown below. The registers of interest with the pertinent signals are highlighted in yellow. The instructions are shown correctly across the top on the "instr" bus, while the registers $s0 to $s3 (16 to 19) show the values indicated in the comments.

A zoomed view of the registers of the waveform are alos provided with the signals of intrest highlighed in yellow.

![alt text](https://raw.githubusercontent.com/IanGoodbody/ECE281_CE5/master/ori_waveform.jpg "ORI waveform")

![alt text](https://raw.githubusercontent.com/IanGoodbody/ECE281_CE5/master/ori_waveform2.jpg "ORI waveform")

Documentation: None
