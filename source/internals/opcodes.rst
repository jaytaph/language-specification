#######
Opcodes
#######

Saffire opcode structure is based on Python opcodes, yet they are not compatible. It is not possible to run Saffire's
bytecode on a Python virtual machine and vice versa.

------------------
Reading the tables
------------------
Each table has 3 values, the opcode hexidecimal number, the name of the opcode and it's description. The description is
short and uses a few abbreviations:

IP
    The instruction pointer. It is always between 0 and the size of the current codeblock and points to the next
    opcode that is currently being executed.
SP
    Stackpointer. Points to the last stored value on the stack. Since stacks grow downwards, SP-0 points to the current
    value, SP-1 points to the previous stored value etc. A stack pointer is always between the current codeblock
    stack-size and 0.
Push
    Adds an object onto the stack (thus **decreasing** the stack pointer). Adding objects onto a full stack will result
    in an error.
Pop
    Removes an object from the stack (thus **increasing** the stack pointer). Popping object from an empty stack will
    result in an error.
OP
    Operand. OP+0 means it's the first operand OP+1 the second etc. An operator is a 2 byte numerical value ranging from
    0x0000 to 0xFFFF
vars
    A table that contains objects that are currently used. Each codeblock has a local vars table and a pointer to the
    global scope vars table.
names
    A table which holds all the names of the variables used in the codeblock. Starts at index 0.
const
    A table which holds objects that are used in the codeblock. Starts at index 0.



------------------------
Opcodes without operands
------------------------
These opcodes do not take any additional operands.

======     ====================     ==========================================
Opcode     Value                    Description
======     ====================     ==========================================
00h        STOP                     Will end the VM execution.

01h        POP_TOP                  Removes (pops) SP-0
02h        ROT_TWO                  Rotates (or swaps) SP-0 with SP-1.
03h        ROT_THREE                Rotates the last three items on the stack, SP-0 becoming SP-2.
04h        DUP_TOP                  Duplicates SP-0 and pushes it onto the stack SP-0 and SP-1 wil be equal.
05h        ROT_FOUR                 Rotates the last four items on the stack SP-0 becoming SP-3.

09h        NOP                      No operation.

17h        BINARY_ADD               Pops SP-0 and SP-1 values from stack and pushes (SP-0 + SP-1)
18h        BINARY_SUB               Pops SP-0 and SP-1 values from stack and pushes (SP-0 - SP-1)
19h        BINARY_MUL               Pops SP-0 and SP-1 values from stack and pushes (SP-0 * SP-1)
1Ah        BINARY_DIV               Pops SP-0 and SP-1 values from stack and pushes (SP-0 / SP-1)
1Bh        BINARY_SHL               Pops SP-0 and SP-1 values from stack and pushes (SP-0 << SP-1)
1Ch        BINARY_SHR               Pops SP-0 and SP-1 values from stack and pushes (SP-0 >> SP-1)
1Dh        BINARY_AND               Pops SP-0 and SP-1 values from stack and pushes (SP-0 & SP-1)
1Eh        BINARY_OR                Pops SP-0 and SP-1 values from stack and pushes (SP-0 | SP-1)
1Fh        BINARY_XOR               Pops SP-0 and SP-1 values from stack and pushes (SP-0 ^ SP-1)

20h        INPLACE_ADD              Pops SP-0 and SP-1 values from stack and pushes (SP-0 + SP-1)
21h        INPLACE_SUB              Pops SP-0 and SP-1 values from stack and pushes (SP-0 - SP-1)
22h        INPLACE_MUL              Pops SP-0 and SP-1 values from stack and pushes (SP-0 * SP-1)
23h        INPLACE_DIV              Pops SP-0 and SP-1 values from stack and pushes (SP-0 / SP-1)
24h        INPLACE_SHL              Pops SP-0 and SP-1 values from stack and pushes (SP-0 << SP-1)
25h        INPLACE_SHR              Pops SP-0 and SP-1 values from stack and pushes (SP-0 >> SP-1)
26h        INPLACE_AND              Pops SP-0 and SP-1 values from stack and pushes (SP-0 & SP-1)
27h        INPLACE_OR               Pops SP-0 and SP-1 values from stack and pushes (SP-0 | SP-1)
28h        INPLACE_XOR              Pops SP-0 and SP-1 values from stack and pushes (SP-0 ^ SP-1)

0Ah        UNARY_POSITIVE
0Bh        UNARY_NEGATIVE
0Ch        UNARY_NOT
0Dh        UNARY_CONVERT
0Fh        UNARY_INVERT
12h        LIST_APPEND
1Eh        SLICE
1Fh        SLICE+1
20h        SLICE+2
21h        SLICE+3
28h        STORE_SLICE
29h        STORE_SLICE+1
2Ah        STORE_SLICE+2
2Bh        STORE_SLICE+3
32h        DELETE_SLICE
33h        DELETE_SLICE+1
34h        DELETE_SLICE+2
35h        DELETE_SLICE+3
3Ch        STORE_SUBSCR
3Dh        DELETE_SUBSCR
44h        GET_ITER
50h        BREAK_LOOP
51h        WITH_CLEANUP
52h        LOAD_LOCALS
53h        RETURN_VALUE
54h        IMPORT_STAR
55h        EXEC_STMT
56h        YIELD_VALUE
57h        POP_BLOCK
58h        END_FINALLY
59h        BUILD_CLASS

7Eh        USE
7Fh        IMPORT                   Imports class SP-0 from SP-1 and pushes it into the stack
======     ====================     ==========================================

.. attention::
    "Inplace" functionality is not implemented. There is no difference between INPLACE_ADD and BINARY_ADD.


----------------------
Opcodes with 1 operand
----------------------
These opcodes all have 1 operand. The opcodes are distinguished from others by looking at the most significant bit.
If that bit is set to 1, it means it has one operand. IN practice, this means **80h** to **BFh** have the highest bit
set. The next opcode **C0h** has the two highest bit set, and is discussed in the `Opcodes with multiple operands`_
section.

======     ====================     ==========================================
Opcode     Value                    Description
======     ====================     ==========================================
80h        STORE_ID                 Stores SP-0 into a variable. The name of the var is found in names[OP+0]. Can be
81h        LOAD_CONST               Create const[OP+0] object and push onto the stack.
82h        LOAD_ID                  Loads identifier(vars[OP+0]) and push onto the stack. Identifier can be local, global or builtin.

83h        JUMP_FORWARD             The IP will be increased with OP+0.
84h        JUMP_IF_FALSE            if SP-0 cast to boolean returns false, it will increase IP with OP+0.
85h        JUMP_IF_TRUE             if SP-0 cast to boolean returns true, it will increase IP with OP+0.
86h        JUMP_ABSOLUTE            IP will be set to OP+0.

87h        DUP_TOPX                 Duplicate SP-0 OP+0 times onto the stack. DUP_TOP is equal to DUP_TOPX[1].

88h        LOAD_GLOBAL              Loads global identifier(vars[OP+0]) and pushes onto the stack.
89h        STORE_GLOBAL             Stores SP-0 into a global identifier(vars[OP+0])
8Ah        DELETE_GLOBAL            Clears global identifier(vars[OP+0])


81h        DELETE_NAME
82h        UNPACK_SEQUENCE
82h        FOR_ITER
5Fh        STORE_ATTR
60h        DELETE_ATTR
??h        BUILD_DATASTRUCTURE      Creates a datastructure with OP+0 elements. SP-0 points to the name of the
                                    datastructure, while SP-N are the element tuples. Pushes back a datastructure object.
66h        BUILD_TUPLE
67h        BUILD_LIST               Same as BUILD_DATASTRUCTURE, except there is no name of the datastructure pushed
                                    onto the stack. Implies "list" and SP-0 points to the first element to be added.
67h        BUILD_HASH               Same as BUILD_DATASTRUCTURE, except there is no name of the datastructure pushed
                                    onto the stack. Implies "hash" and SP-0 points to the first element to be added.
67h        BUILD_SET                Same as BUILD_DATASTRUCTURE, except there is no name of the datastructure pushed
                                    onto the stack. Implies "set" and SP-0 points to the first element to be added.
69h        LOAD_ATTR
6Ah        COMPARE_OP
6Bh        IMPORT_NAME
6Ch        IMPORT_FROM
77h        CONTINUE_LOOP
78h        SETUP_LOOP
79h        SETUP_EXCEPT
7Ah        SETUP_FINALLY
7Ch        LOAD_FAST
7Dh        STORE_FAST
7Eh        DELETE_FAST
82h        RAISE_VARARGS
83h        CALL_FUNCTION            Calls method OP+0 SP+0 from object SP+1 with OP+1 args starting from SP+2.
84h        MAKE_FUNCTION
85h        BUILD_SLICE
86h        MAKE_CLOSURE
87h        LOAD_CLOSURE
88h        LOAD_DEREF
89h        STORE_DEREF
8Ch        CALL_FUNCTION_VAR
8Dh        CALL_FUNCTION_KW
8Eh        CALL_FUNCTION_VAR_KW
8Fh        EXTENDED_ARG
======     ====================     ==========================================



------------------------------
Opcodes with multiple operands
------------------------------

For future reservations, opcodes with two or more operands are possible. They consist of having the highest bits set
to 1. In effect this means that operands starting from **C0h** to **DFh** are reserved for opcodes with 2 operands
(since these opcodes have the highest 2 bits set). Opcdoes **E0h** to **EFh** have the highest 3 bits set, so they are
reserved for 3 operand opcodes. **F0h** to **FEh** are reserved for opcodes with 4 operands. Opcode **FFh** has
special meaning and is discussed in the `Reserved opcodes`_ section.

==========      ==============================================================
Opcodes         Description
==========      ==============================================================
C0h to DFh      2 operand codes (reserved for future use)
E0h to EFh      3 operand codes (reserved for future use)
F0h to FEh      4 operand codes (reserved for future use)
==========      ==============================================================



----------------
Reserved opcodes
----------------
These opcodes should not be used inside Saffire bytecode. When encountered, the VM will halt execution.

======     ====================     ==========================================
Opcode     Value                    Description
======     ====================     ==========================================
FFh        <reserved>               Reserved for future use. Can be used as a marker to indicate special opcode cases.
======     ====================     ==========================================




--------------
Future opcodes
--------------
Virtual machines should be very simple in setup and this should reflect in the number of opcodes that a virtual machine
could handle. Normally, having a maximum of 256 different opcodes should be more than adequate for even the most complex
operations. Still, a virtual machine, just like a computer processor, can "evolve" overtime and accept even more "high-
end" opcodes. In order to keep the bytecode small, we will keep using only 1 byte opcodes, but have **FFh** reserved for
extensions.

Later, when 1-byte opcodes has proven not to be enough, we can use FFh as a marker that another opcode byte will follow.
For instance, the opcodes **FFh 00h** can indicate a future opcode. If even 2 bytes aren't enough, the **FFh** marker can
be used for even larger sets. **FFh FFh FFh 00h** can be distinguished as a unique opcode in a opcode-set of almost
**4 million** different opcodes. I think in this case it is safe to say: 4 million different opcodes should be enough
for everybody.