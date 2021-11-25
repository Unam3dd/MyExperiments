# How resolve external symbols manualy

#### Podalirius and I did some research a few weeks ago on how programs like objdump do to solve the functions of a dynamically linked binary without mapping it to memory or debugging it. It's very simple and useful to understand what operations it does to understand all of this you need to know about PLT and GOT if you don't know anything about it I invite you to look at previous posts

#### when a program is dynamically linked it is impossible to know the addresses of the functions in advance because they are resolved at runtime, this is simple to understand because the addressing of a dynamically linked binary is relative the advantage of having a dynamically linked binary is that it is much lighter than its static version because the functions and other symbols are not directly written in the code, instead the dynamically linked binary will use an interpreter as well as several address resolution procedures to resolve the symbols of the program, these procedures are also called the GOT and the PLT I don't go into the details of its functioning because there is a post based on it.

#### if we take the following program

```c
#include <stdio.h>

int main(void)
{
    printf("hello");
    puts("world");   
    return (0);
}
```

#### here we can see that the following program calls two functions, "printf" and "puts" its two functions allow to write on the file descriptor 1 also commonly called STDOUT which represents the output of the program, in short the program will display Hello a return to the line then our last word World

```c
0000000000001169 <main>:
    1169:       f3 0f 1e fa             endbr64
    116d:       55                      push   rbp
    116e:       48 89 e5                mov    rbp,rsp
    1171:       48 8d 3d 8c 0e 00 00    lea    rdi,[rip+0xe8c]        # 2004 <_IO_stdin_used+0x4>
    1178:       b8 00 00 00 00          mov    eax,0x0
    117d:       e8 ee fe ff ff          call   1070 <printf@plt>
    1182:       48 8d 3d 82 0e 00 00    lea    rdi,[rip+0xe82]        # 200b <_IO_stdin_used+0xb>
    1189:       e8 d2 fe ff ff          call   1060 <puts@plt>
    118e:       b8 00 00 00 00          mov    eax,0x0
    1193:       5d                      pop    rbp
    1194:       c3                      ret
    1195:       66 2e 0f 1f 84 00 00    nop    WORD PTR cs:[rax+rax*1+0x0]
    119c:       00 00 00
    119f:       90                      nop
```

#### once our binary is compiled we will retrieve via objdump the main symbol that corresponds to our entrypoint in our C program

#### now let's take a closer look at the opcodes, the ones we are interested in for this article are the ones starting with e8 because e8 determines a call
http://ref.x86asm.net/coder64.html#xE8

#### we notice that at offset 117d we have a call to the printf function except that if you still have a good view you have surely noticed that the relative address does not correspond to the opcodes finally it is very simple to solve if you know a little bit about binary operators it shouldn't be too complicated

```c
117d:       e8 ee fe ff ff          call   1070 <printf@plt>
```

#### if you take the 4 bytes that represent the relative address you add this to its current offset plus 5 bytes because the call instruction is 5 bytes

#### so the formula is (relative address + offset + 5) followed by an AND operation that will allow us to retrieve only the bits that interest us to retrieve the offset of the PLT stub

#### so if we apply this formula now

```python
py -3 -c "print(hex((0xfffffee + 0x117d + 5) & 0xffffff0))"
```
#### we get 0x1070 which corresponds to our first PLT stub entry here is a nice little technique, now another tip that should help you too

#### if you have ever tried to write a program to disassemble or solve PLT symbols you must have noticed that we can only get their position in the GOT because the role of the plt stub is to make a shift to find it in the GOT it does its job well OK but how to find that printf belongs to such PLT stub
### and well once again it's well thought out the order of the symbols is the same as the PLT stubs which means that if you find for example the symbol printf in first it means that the first stub at the location of the section .plt.sec will be printf@plt and as each PLT stub is 16 bytes long if your second symbol is puts you just have to point your variable on the section .plt.sec + 32 bytes and so on
