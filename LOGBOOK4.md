# **Week #4**

### **SEEDs Lab**
Environment Variable and Set-UID Program Lab

### **Task 1 - Manipulating Environment Variables** 

- Used printenv/env to print out the environment variables. 

**TERMINAL PRINT ON TASK 1**

![Terminal print of task 1](images/task1.png)


We also used the **printenv PWD** command and got the following output: 

![Terminal print of task 1.1](images/task1.1.png)

### **Task 2 - Passing Environment Variables from Parent Process to Child Process**

- We compiled the program given **(gcc myprintenv.c -o cPrint.out gcc myprintenv.c -o pPrint.out)** into two binaries, each differing on which process would print the environment variables (parent or child)

- Run both programs and directed their output to different files. The output was the environment variables they received

- Using the diff command as mentioned on step 3, we concluded that the output was the same for both programs

- **Conclusion:** The environment variables for a parent process and its child process, created through fork with default configuration, are the same

**Program given:** 

``` c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

extern char **environ;
void printenv()
{
  int i = 0;
  while (environ[i] != NULL) {
      printf("%s\n", environ[i]);
      i++;
  }
}

void main()
{
  pid_t childPid;
  switch(childPid = fork()) {
    case 0:  /* child process */
      printenv();          
      exit(0);
    default:  /* parent process */
      //printenv();       
      exit(0);
  }
}
```
**TERMINAL PRINT ON TASK 2**

![Terminal print of task 2](/images/task2.png)


### **Task 3 - Environment Variables and execve()**

- In this task we compiled the program given **(myenv.c)** . This program starts another process through **execve** and prints its environment variables. 

- We did not found any output using the **execve("/usr/bin/env", argv, NULL)**  We think that the reason for this happening is because 'execve' function starts a process and associates to it the array of environment variables given to it in the third argument. As this argument was a NULL pointer, the new process has no environment variables.

- As soon as we changed to **execve("/usr/bin/env", argv, environ)** we got we got the the parent's process environment variables as expected.


``` c
#include <unistd.h>

extern char **environ;
int main()
{
  char *argv[2];

  argv[0] = "/usr/bin/env";
  argv[1] = NULL;
  execve("/usr/bin/env", argv, environ);  

  return 0 ;
}
```

**TERMINAL PRINT ON TASK 3**

![Terminal print of task 3](/images/task32.png)



### **Task 4 - Environment Variables and system()**
