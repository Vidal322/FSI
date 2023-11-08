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






