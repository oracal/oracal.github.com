---
comments: true
date: 2011-02-13 00:06:12
layout: post
slug: explorer-16-lcd
title: Explorer 16 LCD
wordpress_id: 55
categories:
- Microcontroller Programming
tags:
- Microchip
- Microcontroller Programming
- PIC
---

As a follow up to my previous article on the Explorer 16 board we'll look at displaying a string on the LCD screen supplied on the board in a very similar manner to the serial connection function. In fact we're going to use the standard output of the c standard library to be able to output to the serial port or the LCD screen. The LCD screen is a very important peripheral for displaying useful  information and for quickly debugging programs and doesn't require a  connection to a computer, which is handy.

<!-- more -->

Another important point to mention is the fact that we are going to use the Parallel Master Port (PMP) peripheral, new to the PIC24F series PIC. This module has made connecting to many different types of parallel interface much easier and so is useful when connecting to the  LCD.

First of all create a header file called "lcd.h", this will include some useful definitions and function prototypes.

{% codeblock lang:c %}
#include <p24fj128ga010.h>

// define some useful constants

// access data register
#define LCDDATA 1
// access command register
#define LCDCMD  0
// PMP data buffer
#define PMDATA  PMDIN1

void initLCD();         // initialise the LCD
char readLCD(int addr); // read from the LCD

// define some macros that make life a bit easier

// check if LCD busy
#define busyLCD() readLCD( LCDCMD) & 0x80
// check address of LCD position
#define addrLCD() readLCD( LCDCMD) & 0x7F
// read what is on the LCD
#define getLCD()  readLCD( LCDDATA)
// set cursor position
#define setLCDC( a) writeLCD( LCDCMD, (a & 0x7F) | 0x80)

void writeLCD( int addr, char c);    // write to LCD at particular address

void putLCD(char d);    // send a character to be displayed on screen{% endcodeblock %}

The comments pretty much explain the different definitions and macros, the real explanation will be needed for the actual implementation of the functions.

{% codeblock lang:c %}// check if LCD busy
#define busyLCD() readLCD( LCDCMD) & 0x80
// check address of LCD position
#define addrLCD() readLCD( LCDCMD) & 0x7F
// read what is on the LCD
#define getLCD()  readLCD( LCDDATA)
// set cursor position
#define setLCDC( a) writeLCD( LCDCMD, (a & 0x7F) | 0x80){% endcodeblock %}

The four definitions above allow us to check if the LCD is busy,  check the address, get current display and moving the cursor using a  combination of reading the LCD data, checking for the corresponsding  data and writing to the LCD command register .

Again we have an initialisation function which sets up both the PMP and the LCD screen. The LCD is quite a slow electronic component by micro controller standards so we need to put in a few delays during it's initialisation. This is accomplished by using the Timer1 module on the PIC24F. After the initialisation we can actually physically check if the LCD is ready to use, rather than putting in our own delays.

In the comments I've put in important information about the specific registers being set, but I suggest you look at the data sheet to see what exactly is happening.

{% codeblock lang:c %}#include <p24fj128ga010.h>
#include "lcd.h"

// initialise the LCD
void initLCD( void)
{
	// PMP initialization
	PMCON = 0x83BF;             // Enable the PMP
	PMMODE = 0x3FF;             // Master Mode 1
	PMAEN = 0x0001;             // PMA0 enabled

	// initialise TMR1
	T1CON = 0x8030;             // Fosc/2, prescaled 1:256, 16us/tick

	// wait for >30ms
	TMR1 = 0; while( TMR1<2000);// 2000 x 16us = 32ms

	//initiate the HD44780 display 8-bit init sequence
	PMADDR = LCDCMD;            // command register
	PMDATA = 0b00111000;        // 8-bit interface, 2 lines, 5x7
	TMR1 = 0; while( TMR1<3);   // 3 x 16us = 48us

	PMDATA = 0b00001100;        // display ON, cursor off, blink off
	TMR1 = 0; while( TMR1<3);   // 3 x 16us = 48us

	PMDATA = 0b00000001;        // clear display
	TMR1 = 0; while( TMR1<100); // 100 x 16us = 1.6ms

	PMDATA = 0b00000110;        // increment cursor, no shift
	TMR1 = 0; while( TMR1<100); // 100 x 16us = 1.6ms
}
{% endcodeblock %}

Now we create a function that can read from the data register of the LCD, this will allow us to check things like current position and what is currently being display etc.

{% codeblock lang:c %}
// read from the LCD
char readLCD( int addr)
{
	int dummy;
	while( PMMODEbits.BUSY);    // wait for PMP to be available
	PMADDR = addr;              // select the command address
	dummy = PMDATA;             // initiate a read cycle, dummy read
	while( PMMODEbits.BUSY);    // wait for PMP to be available
	return( PMDATA);            // read the status register
}{% endcodeblock %}

The function above reads from the LCD status register and returns it. It does this by first checking to see if the LCD is busy and then computes a dummy read cycle and then checks the data on the PMP once it is no longer busy.

{% codeblock lang:c %}// write to LCD at particular address
void writeLCD( int addr, char c)
{
	while( busyLCD());
	while( PMMODEbits.BUSY);    // wait for PMP to be available
	PMADDR = addr;
	PMDATA = c;
}

// send a character to be displayed on screen
void putLCD( char d)
{
	int z,x;

	// section of code to include special characters such as new line
	switch (d)
	{
		case'\n':
			setLCDC(0x40);
			break;

		case'\r':
			z = addrLCD();
			z=(z+64);
			z=(z&0x40);
			setLCDC(z);
			break;

		case'\t':
			z = addrLCD();
			x= (5 - (z%5));
			setLCDC((z+x));
			break;

		default:
			writeLCD( LCDDATA, d);
			break;
	}
}
{% endcodeblock %}

The two above functions actually write to the LCD screen, writeLCD is the general command that takes in a character and an address and putLCD automatically writes to the LCD at its current position. In the putLCD command we have created cases for when we send special characters to the LCD, these include the newline character '\n', the carriage return character '\r' and the tab character '\t'. Each one performs a specific action using the macros that were defined earlier.

Again we need to re-implement the write function in the c standard library. This time we are going to have any output to stdout going to the LCD screen and any output to stderr going to the serial port. All we do is to use the other cases in the switch statement inside the write function to do different things, as follows:

{% codeblock lang:c %}#include <p24fj128ga010.h>
#include <stdio.h>
#include "conu2.h"
#include "lcd.h"

int write(int handle, void *buffer, unsigned int len)
{
	int i;

	switch (handle)
	{
		case 0:

		// case for stdout outout
		case 1:
			for (i = len; i; --i)
				putLCD( *(char*)buffer++);
			break;

		// case for stderr outout
		case 2:
			for (i = len; i; --i)
				putU2( *(char*)buffer++);
			break;

		default:
			break;
	}

	return(len);
}{% endcodeblock %}

To show our library of functions actually work we can create the following main function to test both peripherals.

{% codeblock lang:c %}#include <p24fj128ga010.h>
#include "conU2.h"
#include <stdio.h>
#include "lcd.h"

int main()
{
	initLCD();
	InitU2();

	printf("hello lcd");

	while(1){

		fprintf(stderr,"hello serial");

	}
}{% endcodeblock %}

The fprintf command allows us to specify the output stream that we want to write to rather than just using printf to output to the default stream of stdout. So in this case stdout is the LCD screen and stderr is the serial port.
