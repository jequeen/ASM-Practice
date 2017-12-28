# Lesson 1 - GPIO

GPIO - General-purpose input/output (GPIO) is a generic pin on an integrated circuit or computer board whose behavior—including whether it is an input or output pin—is controllable by the user at run time. GPIO pins have no predefined purpose, and go unused by default.

*Part I:*

0x20200000 happens to be the address of the GPIO controller

ASM loads the address of GPIO in the r0 register:
```
ldr r0,=0x20200000
```

*Part 2: Enabling Output*

```
mov r1,#1
lsl r1,#18
str r1,[r0,#4]
```

These commands enable output to the 16th GPIO pin. (This is the pin in this particular chip that is connected to the LED

- The value in r1 is needed to enable the LED pin. First line puts 1 into r1.
- The mov command is faster than the ldr command, because it does not involve a memory interaction, whereas ldr loads the value we want to put into the register from memory
- The second instruction is lsl or logical shift left. This means shift the binary representation for the first argument left by the second argument. In this case this will shift the binary representation of 110 (which is 12) left by 18 places (making it  0b1000000000000000000=0d262144).
- Finally the str 'store register' command stores the value in the first argument, r1 into the address computed from the expression afterwards. The expression can be a register, in this case r0, which we know to be the GPIO controller address, and another value to add to it, in this case #4. This means we add 4 to the GPIO controller address and write the value in r1 to that location. This happens to be the location of the second set of 4 bytes

```
/*
The manual says that there is a set of 24 bytes in the GPIO controller, which determine the settings of the GPIO pin. The first 4 relate to the first 10 GPIO pins, the second 4 relate to the next 10 and so on. There are 54 GPIO pins, so we need 6 sets of 4 bytes, which is 24 bytes in total. Within each 4 byte section, every 3 bits relates to a particular GPIO pin. Since we want the 16th GPIO pin, we need the second set of 4 bytes because we're dealing with pins 10-19, and we need the 6th set of 3 bits, which is where the number 18 (6×3) comes from in the code above.
*/
```

*Part 3: Sending GPIO a message*

```
mov r1,#1
lsl r1,#16
str r1,[r0,#40]
```

- The first puts a 1 into r1 as before
- The second shifts the binary representation of this 1 left by 16 places. Since we want to turn pin 16 off, we need to have a 1 in the 16th bit of this next message (other values would work for other pins).
- Finally write it out to the address which is 0d40 added to the GPIO controller address, which happens to be the address to write to turn a pin off (28 would turn the pin on).

*Part 4: Tell the processor what to do instead of crashing*

```
loop$: 
b loop$

```

- The first line here is not a command, but a label. It names the next line loop$. This means we can now refer to the line by name. This is called a label. Labels get discarded when the code is turned into binary, but they're useful for our benefit for referring to lines by name, not number (address). By convention we use a $ for labels which are only important to the code in this block of code, to let others know they're not important to the overall program
- The b (branch) command causes the next line to be executed to be the one at the label specified, rather than the one after it. Therefore, the next line to be executed will be this b, which will cause it to be executed again, and so on forever
-  Thus the processor is stuck in a nice infinite loop until it is switched off safely
- The new line at the end of the block is intentional. The GNU toolchain expects all assembly code files to end in an empty line, so that it is sure you were really finished, and the file hasn't been cut off. If you don't put one, you get an annoying warning when the assembler runs.