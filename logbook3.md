## Task 1

On this task we've learned we can use printenv or env to display some environment variables followed by the name of a particular variable if we want to know more about it.
We can also use "export" and "unset" to set or unset some variables.

## Task 2

By running the myprintenv.c twice and putting the respective outputs in different files we came to the conclusion that the child process inherits the parents' environment variables.

## Task 3

We concluded that by running execve() ,by default,the new program will not inherit the calling process environment variables, howevver if you run with environ in the third argument it inherits all of the calling process env. variables.

## TASK 4

By running the given code we concluded that by running the system() function the environment variables of the calluing process will be passed on to the new program.

## TASK 5

After setting the said environment variables using export() and running the program which then forked a child process, we concluded that the set environment variables that we'd changed weren't inherited.   **doubt**

## TASK 6

Since the system() function is being called from a setuid process the shell program will drop the effective user privilleges preventing the execution of any malicious code


