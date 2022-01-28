# E32C
This is a pipelined version of the E32 RISC-V system, with a very shallow pipeline depth.
Current version has only two pipeline stages; FETCH(+STORE+LOAD) and EXECUTE(+DECODE+WBACK).

This implementation is as simple as possible, and conforms to the RV32I model of RISC-V ISA.

At this time, this system has good timing closure at 150MHz base clock speed (CPU+Bus on same clock).
In the future this will be changed to set up the bus at a different speed than the rest of the system.

# FETCH unit
The FETCH unit is a self-stalling stage. It will repeatedly read instructions starting from the
reset vector, until it sees a branch, jump, load or store instruction, after which it will go
to a STALL mode.

In this mode, FETCH unit will wait for either a read or write flag to be set, or the new program
counter to be calculated, after which it will resume normal operation.

The program counter and the memory bus are tied to the FETCH unit, as it does all the memory access.

At the output side of FETCH unit there's a FIFO that stores the read instruction and its program
counter (to be used for some relative offset calculations by the EXECUTE unit). The FIFO is therefore
64 bits wide, and currently 512 deep (which is a bit overkill but the built-in FIFO on the target device
does not offer a smaller choice)

Since this unit does all memory access, it's considered a unified FETCH+STORE+LOAD unit.

# EXECUTE unit
The EXECUTE unit is the main CPU flow controller.

It pulls one instruction from the FETCH unit's output fifo, and handles the ALU, memory address calculation,
branch address calculation and register writeback operations.

EXECUTE unit will unblock the self-stalling FETCH unit once it has enough information to let it resume
operations.

EXECUTE unit can be further decoupled from the FETCH unit by changing the FETCH unit's FIFO to a dual-clock
version, in which case the EXECUTE unit can be set to a faster clock speed as required.

# ROM
Default ROM image comes with Bruno Levy's port of tinyraytracer: https://github.com/ssloy/tinyraytracer
It is compiled with RV32I instruction set and does not use any multipliers/dividers/FPU.

To see the output, you'll need to attach a terminal software capable of interpreting RGB color codes to
the COM port your FPGA board is connected to, and set it to 115200 baud, 8 bits, 1stop bit, no parity.

Default output assumes a terminal window width of 80 columns.

# TODO
The plan is to gradually bring back all the features of E32B into E32C, changing the architecture as needed
to either keep or expand upon the pipelined operation.

# FPGA board used in the design
The system is developed on a Nexys Video board with an A7-200T but since the first versions' current resource
utilization is so small (1230 LUTs + 631 FFs), and doesn't depend on anything except three FPGA pins (one for
clock, two for UART), it should happily run on any smaller FPGA.
