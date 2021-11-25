# How resolve symbols manualy

#### Podalirius and I did some research a few weeks ago on how programs like objdump do to solve the functions of a dynamically linked binary without mapping it to memory or debugging it. It's very simple and useful to understand what operations it does to understand all of this you need to know about PLT and GOT if you don't know anything about it I invite you to look at previous posts

#### when a program is dynamically linked it is impossible to know the addresses of the functions in advance because they are resolved at runtime, this is simple to understand because the addressing of a dynamically linked binary is relative the advantage of having a dynamically linked binary is that it is much lighter than its static version because the functions and other symbols are not directly written in the code, instead the dynamically linked binary will use an interpreter as well as several address resolution procedures to resolve the symbols of the program, these procedures are also called the GOT and the PLT I don't go into the details of its functioning because there is a post based on it.

