; Tech ID: 12676184
; Author: Matthew Romleski
; Program for the XMEGA_A1U Xplained pro kit which toggles the LED on the board when an
; interrupt is recieved through a button on PortQ pin 3. LED is on by default.

	.dseg	
	.def	temp = r16

	.cseg
	.org	0x00
	rjmp	start
	.org	PORTC_INT0_vect ; Set up the interrupt vector table.
	rjmp	PF_INT0_ISR ; ^^
	.org	0xF6

start:
    ldi		temp, low(RAMEND) ; Loads the stack pointer.
	sts		CPU_SPL, temp ; ^^
	ldi		temp, high(RAMEND) ; ^^
	sts		CPU_SPH, temp ; ^^
	
	ldi		temp, 0x08 ; Selects the 3rd pin in a mask.
	sts		PORTQ_DIRSET, temp ; Sets the 3rd pin of port Q as output.
	sts		PORTC_DIRCLR, temp ; Sets the 3rd pin of port C as input.

	ldi		temp, 0x01 ; Sets the 3rd pin of port C for disabled slew limit,
	sts		PORTC_PIN3CTRL, temp ; non-inverted, totem pole, and rising edge detection.

	ldi		temp, 0x08 ; Selects 3rd pin.
	sts		PORTC_INT0MASK, temp ; Sets the 3rd pin to use interrupt 0.
	
	ldi		temp, 0x03 ; Loads the code for high interrupt priority.
	sts		PORTC_INTCTRL, temp ; Sets port C to high priority.

	ldi		temp, 0x04 ; Enable high priority interrupts.
	sts		PMIC_CTRL, temp ; ^^

	sei ; Enable interrupts globally.

foreverSetup:
	ldi		temp, 0x00 ; Sets the LED to be on be default.

forever:
	sts		PORTQ_OUT, temp
	jmp		forever ; Infinite loop.


PF_INT0_ISR:
	ldi		temp, 0x00 ; Clear the interrupt flag.
	sts		PORTC_INTFLAGS, temp

	lds		temp, PORTQ_OUT ; Reads the current state of PORTQ.

	andi	temp, 0x08 ; Gets only the value of the 3rd pin.

	cpi		temp, 0x08 ; Checks if the pin is high (LED is off).
	breq	toggleOn ; If it is, toggle the pin low (turn LED on).
	rjmp	toggleOff ; Otherwise, the pin is low, and needs to be toggled high (turn LED off).

toggleOn:
	ldi		temp,0x00 ; Set the 3rd pin to low (LED on).
	sts		PORTQ_OUT,temp ; ^^
	rjmp	return

toggleOff:
	ldi		temp,0x08 ; Set the 3rd pin to high (LED off).
	sts		PORTQ_OUT,temp ; ^^
	rjmp	return

return: 
	reti