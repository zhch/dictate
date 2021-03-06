### Interrupts
分为3类：
#### IRQ (Hardware Interrupts)
An interrupt request (IRQ) is a request for service, sent at the hardware level. Interrupts can be sent by either a dedicated hardware line, or across a hardware bus as an information packet (a Message Signaled Interrupt, or MSI).

#### System Call(Software Interrupts)

#### Exceptions (Traps)

### IDT
the CPU has a table called the IDT, which is a vector table setup by the OS, and stored in memory. There are 256 interrupt vectors on x86 CPUs, numbered from 0 to 255 which act as entry points into the kernel. 

one should refrain from having interrupts of different types coming in on the same vector. Common practice is to leave the first 32 vectors for exceptions, as mandated by Intel. However you partition of the rest of the vectors is up to you.

### EOI
An End Of Interrupt (EOI) is a signal sent to a Programmable Interrupt Controller (PIC) to indicate the completion of interrupt processing for a given interrupt. An EOI is used to cause a PIC to clear the corresponding bit in the In Service Register (ISR), and thus allow more interrupt requests of equal or lower priority to be generated by the PIC.

### Spurious IRQ
Spurious IRQs:
When an IRQ occurs, the PIC chip tells the CPU (via. the PIC's INTR line) that there's an interrupt, and the CPU acknowledges this and waits for the PIC to send the interrupt vector. This creates a race condition: if the IRQ disappears after the PIC has told the CPU there's an interrupt but before the PIC has sent the interrupt vector to the CPU, then the CPU will be waiting for the PIC to tell it which interrupt vector but the PIC won't have a valid interrupt vector to tell the CPU.

To get around this, the PIC tells the CPU a fake interrupt number. This is a spurious IRQ. The fake interrupt number is the lowest priority interrupt number for the corresponding PIC chip (IRQ 7 for the master PIC, and IRQ 15 for the slave PIC).

There are several reasons for the interrupt to disappear. In my experience the most common reason is software sending an EOI at the wrong time. Other reasons include noise on IRQ lines (or the INTR line).

### From the keyboard's perspective
When a key is pressed, the keyboard controller tells a device called the PIC, to cause an interrupt. Because of the wiring of keyboard and PIC, IRQ #1 is the keyboard interrupt, so when a key is pressed, IRQ 1 is sent to the PIC. Translate the IRQ number into an interrupt vector (i.e. a number between 0 and 255) for the CPU's table.

The OS is supposed to handle the interrupt by talking to the keyboard, via in and out instructions, asking what key was pressed, doing something about it (such as displaying the key on the screen, and notifying the current application that a key has been pressed), and returning to whatever code was executing when the interrupt came in. Indeed, failure to read the key from the buffer will prevent any subsequent IRQs from the keyboard.


### From the PIC's perspective
... ... ... ...


### From the CPU's perspective
Every time the CPU is done with one machine instruction, it will check if the PIC's pin has notified an interrupt. If that's the case, it stores some state information on the stack (so that it can return to whatever it is doing currently, when the INT is done being serviced by the OS) and jumps to a location pointed to by the IDT. The OS takes over from there. The current program can, however, prevent the CPU from being disturbed by interrupts by means of the interrupt flag (IF in status register). As long as this flag is cleared, the CPU ignores the PIC's requests and continues running the current program. Assembly instructions cli and sti can control that flag.

### good docs
* https://github.com/GiantVM/doc/tree/master/interrupt_and_io
* https://cs.uwaterloo.ca/~brecht/servers/apic/SMP-affinity.txt
* https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/performance_tuning_guide/s-cpu-irq
* https://github.com/torvalds/linux/blob/master/Documentation/IRQ-affinity.txt
