Exceptions
	an exception is an event that occurs during the execution of a program that disrupts the normal flow of instructions. Exceptions can be triggered by a variety of factors, such as a divide-by-zero error, an illegal instruction, or an attempt to access memory that is not allocated to the program.
	solution:
	when an exception occurs, the processor typically saves the current state of the program and transfers control to a special routine known as an exception handler, which is responsible for dealing with the exception and returning control to the main program. Exception handling is an importand aspect of computer architecture as it allows the system to gracefully handle errors and unexpected situations.
Reasons for exception:
	- External reasons (asynchronous events):
		  incoming data, device ready, mouse movement, ...
		  1. RESET: Reset of the processor, e.g., triggered by a button, power supply, watch dog timer, ..
		  2. HALT: Stop the execution of the processor, e.g., to avoid access conflicts on the system bus during DMA (direct memory access)
		  3. ERROR: Call of an error handler routine, e.g., due to bus errors
		  4. Interrupt:
		     - <mark style="background: #FFB8EBA6;">Interrupt</mark> request triggered by an external device, e.g., to announce incoming data of an input device.
		     - 2 types: maskable/non maskable (NMI)
	- Internal reasons (synchronous events):
		system calls, debugging, change of privilege, ...
		1. Software Interrupts:
			INT instruction in the program triggers an interrupt (system calls, debugging, ...)
		2. Traps:
			Exceptions caused by internal events, e.g., overflow, division by zero, stack overflow, ...
Subroutines
	In the occurance of exceptions, subroutines can be used as part of the exception handling process. A subroutine is a block of code that performs a specific task
	and can be called upon by the main program or other subroutines. 
Interrupt
	An interrupt is an event that temporarily suspends the normal execution of a program and transfers control to a specific routine in order to service some external event such as a keyboard press, a mouse movement, or a network packet arrival
ISR
	Handling of exceptions requires specialized routines (Interrupt Service Routine, ISR).
	ISR is a subroutine in that is executed in response to an interrupt.
	It's a software routine that is automatically executed by the processor when an interrupt occurs.
	It's responsible for determining the source of the interrupt, and taking appropriate action to handle it.
	Once it has completed its task, it typically roturns control to the main program, allowing the program to resume normal execution.
	The advantage of using ISR is that it allows the system to respond quickly to external events, without having to wait for the main program to check for the event. It also allows the system to perform multiple tasks simultaneously by handling multiple interrupts.
ISR vs Subroutines:
	An ISR has the same structure as a subprogram, but there are also some differences.
	1. Activation:
		a Subroutine is being activated by `call subroutine`, but ISR can be activated by `INT` instruction or hardware activation.
	2. Return after completion: 
	    Sub: `RET` instruction
	    ISR: `RETI` instruction
	3. calculation of starting address:
		Sub: Starting address of called subroutine written in calling program
		ISR: Starting address of called ISR determined via interrupt table
	4. Saving Status:
		Sub: saves only PC on a stack
		ISR: saves PSW as well as PC on a stack
	5. Cause of execution:
		Sub: the processor always executes subroutine calls as programmed.
		ISR: The processor executes ISR only if triggered and the interrupt enable bit in the PSW is set.
Calculation of the start address of an ISR
	
	![[ISA.png]]
Typical steps of an ISR
	1. Interrupt activation
	2. Finalize the instruction currently in execution
	3. Check, if software interrupt or internal/external hardware interrupt
	4. Check if Interrupt Enable bit is set then allow interrupt
	5. If it is a hardware interrupt: find source of interrupt, activate INTA (interrupt acknowledge)
	6. Save PSW and PC on stack
	7. Reset Interrupt Enable bit to avoid an additional interrupt in this stage
	8. Calculate start address of ISR (e.g. based on the interrupt vector table) and load it into the PC
	9. Execute the Interrupt Service Routine:  
		- Push the used register on stack  
		- Set the Interrupt Enable bit to allow other interrupts (i.e. interrupts can interrupt interrupts!)  
		- Do the real work of the ISR  
		- Pop the registers from stack  
		- Return from interrupt handling using the IRET instruction
	10. Restore PSW and PC and continue with the interrupted program
	Be aware: if the ISR is too large it blocks the computer
Interrupt Vector Table (IVT)
	The IVT is a table of memory addresses that the CPU uses to determine the location of the ISR for a specific interrupt. It is usually located in the memory at a fixed address, and each entry in the table corresponds to a specific interrupt. When an interrupt occurs, the CPU looks up the corresponding entry in the IVT to find the address of the ISR that should be executed.
	Typically, located at a well-known address, e.g., in ROM (starting at address 0000:0000 for 80X86 processors
Multiple Interrupts
	It's a situation where a higher-priority interrupt occurs while the CPU is handling a lower-priority interrupt.
	When the higher-priority interrupt occurs, the CPU stops executing the current ISR and saves the state of the system. It then looks up the address of the ISR for the higher-priority interrupt in the IVT and starts executing it. Once the higher-priority ISR has finished executing, the CPU restores the state of the system and resumes executing the lower-priority ISR from where it left off.
	This process is called "<mark style="background: #FFB8EBA6;">nesting</mark>" of interrupts, and it allows the system to 
	handle multiple interrupts simultaneously and respond to high-priority events as soon as possible.
	
	![[ISA2.png]].
Polling
	Polling, on the other hand, is a method where the CPU periodically checks the status of devices to see if they need to be serviced. The CPU repeatedly checks the devices, and if one has an event that needs to be handled, it will do so.
	Examples of Polling:  
	 • A program that needs to read the current temperature of a sensor in a regular interval, repeatedly reads the value of the sensor at fixed intervals of time.  
	 • A program that needs to check if a button has been pressed, repeatedly reads the state of the button in a loop.
Interrupt vs Polling
	[G4G](https://www.geeksforgeeks.org/difference-between-interrupt-and-polling/)
Handling of multiple interrupt sources
	Cyclic polling of interrupt sources by the interrupt controller (interrupt flag for each source in a status register of the controller).
	If the interrupt flag for a component is set, then stop cyclic polling and start ISR for the source.
	Several new/additional interrupt requests possible during or after the execution of an ISR. hence, two alternative ways of treating new/additional interrupts
Types of polling
	1. fair processor sharing
		Continue cycling polling at the interrupt source following the last served source. all interrupt sources have an equal chance of being served („fair“ processor sharing)
		![[Pasted image 20230119112524.png]]
	2. prioritizing
		Cyclic polling always starts at a pre-determined first source  Different sources automatically get different priorities. Polling favors components with higher priority.
		![[Pasted image 20230119112541.png]]

Daisy chaining using Hardware
	___
	Daisy chain refers to a chain of devices connected in such a way that each device can signal an interrupt to the next device in the chain. This allows multiple devices to share a single interrupt line, rather than each device having its own interrupt line.  
	___
	When an interrupt occurs on one of the devices in the chain, it signals the interrupt to the next device in the chain, and so on, until the interrupt reaches the CPU. The interrupt signal is passed through the chain of devices in a specific order, with each device checking if the interrupt is meant for itself before passing it on to the next device.  
	When the CPU receives the interrupt signal, it looks up the address of the ISR for the interrupt in the IVT and starts executing it. Once the ISR has finished executing, the CPU sends an acknowledge signal to the devices in the chain to indicate that the interrupt has been handled.
	___
	Use specialized hardware for <mark style="background: #FFB8EBA6;">prioritization</mark> and <mark style="background: #FFB8EBA6;">identification</mark> of interrupts.  
	Example: Chaining of interrupt sources to a priority chain (Interrupt Daisy Chain).  
	Each source for an interrupt uses dedicated hardware for connecting with a successor and predecessor (decentralized prioritization)  
	The first source in the chain has automatically the <mark style="background: #FFF3A3A6;">highest priority</mark>.  
	The priority of the other sources depend on the <mark style="background: #FFF3A3A6;">position</mark> 
	in the chain.
	___
	The daisy chain method allows multiple devices to share a single interrupt line, which can be more cost-effective and space-saving than having a separate interrupt line for each device. It can also simplify the wiring of the system and improve the performance by reducing the number of interrupt lines that the CPU needs to handle.
	___
Advantages of Daisy chaining
	1. Cost-effective: Daisy chaining allows multiple devices to share a single interrupt line, which can be more cost-effective than having a separate interrupt line for each device. This can be especially useful in situations where there are a large number of devices that need to be connected.  
	2. Space-saving: Daisy chaining can also save space as it requires fewer cables and connections than having a separate interrupt line for each device. This can be especially useful in situations where there is limited space for cables and connections.  
	3. Simplified Wiring: Daisy chaining can simplify the wiring of the system as it requires fewer connections, this can be especially useful in situations where a large number of devices need to be connected.  
	4. Improved Performance: Daisy chaining can improve the performance of the system by reducing the number of interrupt lines that the CPU needs to handle. This can allow the CPU to handle interrupts more efficiently and respond to high-priority events more quickly.  
	5. Scalability: Daisy chaining can be scalable, it can be easily extended to include more devices as needed.
Sources:
	[Difference between Interrupt and Polling](https://www.geeksforgeeks.org/difference-between-interrupt-and-polling/)
	[Difference between Hardware Interrupt and Software Interrupt](https://www.geeksforgeeks.org/difference-between-hardware-interrupt-and-software-interrupt/)
	[Difference between Maskable and Non Maskable Interrupt](https://www.geeksforgeeks.org/difference-between-maskable-and-non-maskable-interrupt/)
	[Difference between Interrupt and Exception](https://www.geeksforgeeks.org/difference-between-interrupt-and-exception/?ref=rp)
	[Purpose of an Interrupt in Computer Organization](https://www.geeksforgeeks.org/purpose-of-an-interrupt-in-computer-organization/?ref=rp)
	[Interrupts](https://www.geeksforgeeks.org/purpose-of-an-interrupt-in-computer-organization/?ref=rp)
	[Interrupts and Exceptions](https://www.geeksforgeeks.org/interrupts-and-exceptions/?ref=gcse)
	[https://www.geeksforgeeks.org/interrupts-and-exceptions/?ref=gcse](https://www.geeksforgeeks.org/difference-between-interrupt-and-exception/?ref=gcse)
	[Priority Interrupts | (S/W Polling and Daisy Chaining)](https://www.geeksforgeeks.org/priority-interrupts-sw-polling-daisy-chaining/?ref=gcse)
	[Return from Subroutine and Return from Interrupt](https://www.geeksforgeeks.org/return-from-subroutine-and-return-from-interrupt/?ref=gcse)
	