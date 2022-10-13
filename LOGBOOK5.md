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


