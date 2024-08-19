# Asynchronous-FIFO

Introduction
FIFO stands for "First In, First Out," and it is a method used for managing data, inventory, or resources. The principle behind FIFO is straightforward: the first item added to a system will be the first one to be removed or processed. This concept is widely applied in various fields, including computing, inventory management, and financial accounting.
In order to understand FIFO design, one needs to understand how the FIFO pointers work. The write pointer always points to the next word to be written; therefore, on reset, both pointers are set to zero, which also happens to be the next FIFO word location to be written. On a FIFO-write operation, the memory location that is pointed to by the write pointer is written, and then the write pointer is incremented to point to the next location to be written.

Similarly, the read pointer always points to the current FIFO word to be read. Again on reset, both pointers are reset to zero, the FIFO is empty and the read pointer is pointing to invalid data (because the FIFO is empty and the empty flag is asserted)


Gray Code Counter
A Gray code counter is a type of counter that uses Gray code, a binary numeral system where two successive values differ in only one bit. Gray codes are named after Frank Gray, who introduced them in 1930. They are often used in digital systems where minimizing errors due to bit changes is important.
It is desirable to create both an n-bit Gray code counter and an (n-1)-bit Gray code counter called a “dual n-bit Gray code counter”. The most common Gray code is a reflected code where the bits in any column except the MSB are symmetrical about the sequence mid-point.

We want the LSBs of the second half to repeat the 4-bit LSBsequence of the first half. Inverting the second MSB of the second half of the 4-bit Gray code will produce the desired 3-bit Gray code sequence in the three LSBs of the 4-bit sequence.

There are no odd-count-length Gray code sequences so one cannot make a 23-deep Gray code. This means that the technique is used to make a FIFO that is 2^n deep.

Gray code counter - Style #1
Dual n-bit Gray code counter

A dual n-bit Gray code counter is a Gray code counter that generates both an n-bit Gray code sequence (described in section 3.2) and an (n-1)-bit Gray code sequence
![image](https://github.com/user-attachments/assets/19009592-e1d2-4537-9d5f-2971d0d5c001)

Gray code counter - Style #2
The FIFO implementation uses the Gray code counter style #2, which actually employs two sets of registers to eliminate the need to translate Gray pointer values to binary values. The second set of registers (the binary registers) can also be used to address the FIFO memory directly without the need to translate memory addresses into Gray codes. The n-bit Gray-code pointer is still required to synchronize the pointers into the opposite clock domains, but the n-1-bit binary pointers can be used to address memory directly.
![image](https://github.com/user-attachments/assets/12b68663-34eb-4a4e-905a-9990597a415c)

Design and Architecture
![image](https://github.com/user-attachments/assets/f548bce8-d408-4007-9677-4c6efb2ee61a)

Fig: FIFO1 partitioning with synchronized pointer comparison

To facilitate static timing analysis of the style #1 FIFO design, the design has been partitioned into the following six Verilog modules with the following functionality and clock domains:

fifo1.v - (see Example 2 in section 6.1) - this is the top-level wrapper-module that includes all clock domains. The top module is only used as a wrapper to instantiate all of the other FIFO modules used in the design.

fifomem.v - (see Example 3 in section 6.2) - this is the FIFO memory buffer that is accessed by both the write and read clock domains. This buffer is most likely an instantiated, synchronous dual-port RAM. Other memory styles can be adapted to function as the FIFO buffer

sync_r2w.v - (see Example 4 in section 6.3) - this is a synchronizer module that is used to synchronize the read pointer into the write-clock domain. The synchronized read pointer will be used by the wptr_full module to generate the FIFO full condition.

sync_w2r.v - (see Example 5 in section 6.4) - this is a synchronizer module that is used to synchronize the write pointer into the read-clock domain. The synchronized write pointer will be used by the rptr_empty module to generate the FIFO empty condition.

rptr_empty.v - (see Example 6 in section 6.5) - this module is completely synchronous to the read-clock domain and contains the FIFO read pointer and empty-flag logic.

wptr_full.v - (see Example 7 in section 6.6) - this module is completely synchronous to the write-clock domain and contains the FIFO write pointer and full-flag logic.

Handling full & empty conditions
The empty flag will be generated in the read-clock domain. The read pointer catches up to the write pointer (including the pointer MSBs). Pointers that are one bit larger than needed to address the FIFO memory buffer are used. In order to efficiently register the rempty output, the synchronized write pointer is actually compared against the rgraynext (the next Gray code that will be registered into the rptr).

assign rempty_val = (rgraynext == rq2_wptr); always @(posedge rclk or negedge rrst_n) if (!rrst_n) rempty <= 1'b1; else rempty <= rempty_val;

Generating full
The full flag will be generated in the write-clock domain. The write pointer catches up to the read pointer (except for different pointer MSBs). The read pointer be synchronized into the write clock domain before doing pointer comparison. Three conditions that are all necessary for the FIFO to be full:

(1) The wptr and the synchronized rptr MSB's are not equal (because the wptr must have wrapped one more time than the rptr).

(2) The wptr and the synchronized rptr 2nd MSB's are not equal

(3) All other wptr and synchronized rptr bits must be equal

assign wfull_val = (wgraynext=={~wq2_rptr[ADDRSIZE:ADDRSIZE-1], wq2_rptr[ADDRSIZE-2:0]}); always @(posedge wclk or negedge wrst_n) if (!wrst_n) wfull <= 1'b0; else wfull <= wfull_val;

Waveform
![image](https://github.com/user-attachments/assets/1a395ede-c4e8-4e97-9f41-af818436a66b)



