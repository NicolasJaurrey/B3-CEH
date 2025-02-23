# Part I : Learn

Dans cette partie, je vous fais (re)découvrir quelques commandes usuelles quand on travaille autour des programmes et des processus.

**Au menu : on dissèque des programmes, et on repère les syscalls qu'ils utilisent.**

## Sommaire

- [Part I : Learn](#part-i--learn)
  - [Sommaire](#sommaire)
  - [1. Anatomy of a program](#1-anatomy-of-a-program)
    - [A. `file`](#a-file)
    - [B. `readelf`](#b-readelf)
    - [C. `ldd`](#c-ldd)
  - [2. Syscalls basics](#2-syscalls-basics)
    - [A. Syscall list](#a-syscall-list)
    - [B. `objdump`](#b-objdump)

## 1. Anatomy of a program

**Un programme est un fichier *exécutable*. C'est à dire que :** 

- c'est un simple fichier
- il est composé de plusieurs sections
  - la section `.text` contient les instructions du programme pour le CPU
  - les autres sections contiennent essentiellement des metadonnées
- il peut être compilé...
  - statiquement : tout est dans le programme
  - dynamiquement : le programme pourra faire appel à des librairies du système
- il est marqué comme étant "exécutable"
  - sur Linux, on donne la permission d'exécution avec `chmod`

Dans cette partie, on va voir quelques outils très usuels pour obtenir des infos sur un programme.

### A. `file`

`file` est une commande uqi permet de déterminer le type d'un fichier.

Ceci ne repose pas du tout sur l'extension du fichier. `file` regarde directement les bits qui composent le fichier pour en déterminer le type. Il se concentre sur les premiers octets du fichiers qui contient généralement des métadonnées suffisantes pour déterminer le type.

🌞 **Utiliser `file` pour déterminer le type de :**

- la commande `ls`

```
[rockynj@RockyNJ ~]$ file /usr/bin/ls
/usr/bin/ls: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=1afdd52081d4b8b631f2986e26e69e0b275e159c, for GNU/Linux 3.2.0, stripped
```

- la commande `ip`

```
[rockynj@RockyNJ ~]$ file /usr/sbin/ip
/usr/sbin/ip: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=77a2f5899f0529f27d87bb29c6b84c535739e1c7, for GNU/Linux 3.2.0, stripped
```

- un fichier `.mp3` que vous aurez téléchargé sur le disque de la VM

```
[rockynj@RockyNJ ~]$ file Enregistrement.mp3 
Enregistrement.mp3: Audio file with ID3 version 2.3.0, contains:MPEG ADTS, layer III, v1, 192 kbps, 48 kHz, JntStereo

```

> Le format des exécutables sous les OS Linux est appelé ELF. ELF est le format qui définit l'ordre des octets dans un programme, le fait qu'il doit être composé de plusieurs sections, comment il doit indiquer les librairies externes dont il a besoin, etc.

### B. `readelf`

`readelf` permet d'obtenir des informations sur un fichier ELF : un exécutable Linux.

De la même façon qu'un fichier texte possède des numéros de ligne quand on l'affiche, si on affiche le contenu d'un programme, chaque ligne est numérotée.

Chaque ligne du programme a donc une adresse, qui est notée en hexadécimal.

`readelf` permet notamment de voir de quelle adresse à quelle adresse se trouve  tell ou telle section.

🌞 **Utiliser `readelf` sur le programme `ls`**

- afficher le *header* du programme
  - il contient toutes les métadonnées principales du programme
  - c'est l'option `readelf -h`

```
  [rockynj@RockyNJ ~]$ readelf -h /usr/bin/ls
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Shared object file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x6b10
  Start of program headers:          64 (bytes into file)
  Start of section headers:          139032 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         30
  Section header string table index: 29
```

- afficher la liste des sections du programme
  - c'est l'option `readelf -S`

```
[rockynj@RockyNJ ~]$ readelf -s /usr/bin/ls

Symbol table '.dynsym' contains 125 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __[...]@GLIBC_2.3 (2)
     2: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
     3: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND cap_to_text
     4: 0000000000000000     0 OBJECT  GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
     5: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
     6: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.3.4 (4)
     7: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND raise@GLIBC_2.2.5 (3)
     8: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND free@GLIBC_2.2.5 (3)
     9: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND _[...]@GLIBC_2.34 (5)
    10: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND abort@GLIBC_2.2.5 (3)
    11: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    12: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    13: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_deregisterT[...]
    14: 0000000000000000     0 OBJECT  GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    15: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    16: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND _exit@GLIBC_2.2.5 (3)
    17: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    18: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    19: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    20: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    21: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    22: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    23: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    24: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    25: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    26: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND c[...]@GLIBC_2.17 (6)
    27: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    28: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    29: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    30: 0000000000000000     0 OBJECT  GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    31: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    32: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    33: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    34: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    35: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    36: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    37: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __[...]@GLIBC_2.4 (7)
    38: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    39: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    40: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@LIBSELINUX_1.0 (8)
    41: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    42: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    43: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    44: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    45: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    46: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    47: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND lseek@GLIBC_2.2.5 (3)
    48: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    49: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    50: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    51: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND ioctl@GLIBC_2.2.5 (3)
    52: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    53: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    54: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND lstat@GLIBC_2.33 (9)
    55: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    56: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    57: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    58: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    59: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    60: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    61: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND dirfd@GLIBC_2.2.5 (3)
    62: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    63: 0000000000000000     0 OBJECT  GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    64: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.3.4 (4)
    65: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    66: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND __gmon_start__
    67: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.14 (10)
    68: 0000000000000000     0 OBJECT  GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    69: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND tzset@GLIBC_2.2.5 (3)
    70: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    71: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    72: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    73: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    74: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    75: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    76: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    77: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    78: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    79: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    80: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    81: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    82: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    83: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    84: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.3.4 (4)
    85: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND statx@GLIBC_2.28 (11)
    86: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    87: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    88: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    89: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    90: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND error@GLIBC_2.2.5 (3)
    91: 0000000000000000     0 OBJECT  GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    92: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    93: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND cap_get_file
    94: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    95: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    96: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND cap_free
    97: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    98: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
    99: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND ge[...]@GLIBC_2.3 (2)
   100: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
   101: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
   102: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND exit@GLIBC_2.2.5 (3)
   103: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
   104: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.3.4 (4)
   105: 0000000000000000     0 NOTYPE  WEAK   DEFAULT  UND _ITM_registerTMC[...]
   106: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@LIBSELINUX_1.0 (8)
   107: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
   108: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
   109: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@LIBSELINUX_1.0 (8)
   110: 0000000000000000     0 OBJECT  GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
   111: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
   112: 0000000000000000     0 FUNC    WEAK   DEFAULT  UND [...]@GLIBC_2.2.5 (3)
   113: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
   114: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __[...]@GLIBC_2.3 (2)
   115: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND __[...]@GLIBC_2.3 (2)
   116: 0000000000000000     0 OBJECT  GLOBAL DEFAULT  UND [...]@GLIBC_2.2.5 (3)
   117: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND [...]@GLIBC_2.3.4 (4)
   118: 00000000000220a0     8 OBJECT  GLOBAL DEFAULT   25 obstack_alloc_fa[...]
   119: 000000000000fe20   297 FUNC    GLOBAL DEFAULT   15 _obstack_newchunk
   120: 000000000000fe00    25 FUNC    GLOBAL DEFAULT   15 _obstack_begin_1
   121: 0000000000010840    55 FUNC    GLOBAL DEFAULT   15 _obstack_allocated_p
   122: 000000000000fde0    21 FUNC    GLOBAL DEFAULT   15 _obstack_begin
   123: 0000000000010910    38 FUNC    GLOBAL DEFAULT   15 _obstack_memory_used
   124: 0000000000010880   136 FUNC    GLOBAL DEFAULT   15 _obstack_free
```

- déterminer à quel adresse commence le code du programme
  - pour rappel, le code est dans la section `.text`
  - vous devriez voir cette adresse dans la sortie de `readelf -S`

  ```
  [rockynj@RockyNJ ~]$ readelf -S /usr/bin/ls | grep .text
  [15] .text             PROGBITS         0000000000004d50  00004d50
  ```

### C. `ldd`

`ldd` est un outil qui permet de manipuler le *dynamic linker* de Linux. Le *dynamic linker* c'est un programme qui s'occupe de trouver les librairies nécessaires quand un autre programme se lance.

**On peut utiliser `ldd` notamment pour visualiser de quelle librairie a besoin un programme donné.**

🌞 **Utiliser `ldd` sur le programme `ls`**

- afficher la liste des librairies que va utiliser `ls` pendant son fonctionnement
```
[rockynj@RockyNJ ~]$ ldd /usr/bin/ls
	linux-vdso.so.1 (0x00007fff3ad8f000)
	libselinux.so.1 => /lib64/libselinux.so.1 (0x00007efff6b64000)
	libcap.so.2 => /lib64/libcap.so.2 (0x00007efff6b5a000)
	libc.so.6 => /lib64/libc.so.6 (0x00007efff6800000)
	libpcre2-8.so.0 => /lib64/libpcre2-8.so.0 (0x00007efff6abe000)
	/lib64/ld-linux-x86-64.so.2 (0x00007efff6bbb000)
```

- déterminer, parmi ces librairies, laquelle est la Glibc

```
[rockynj@RockyNJ ~]$ ldd /usr/bin/ls | grep libc.so
	libc.so.6 => /lib64/libc.so.6 (0x00007f0438400000)
```

> La Glibc est une des librairies les plus importantes au sein d'un système Linux, car elle contient notamment tout le nécessaire pour passer des *syscalls* élémentaires. Si un programme souhaite lire ou écrire dans un fichier par exemple, il aura besoin d'inclure la Glibc.

## 2. Syscalls basics

### A. Syscall list

> Vous pourrez trouver une [liste des syscalls Linux sur un système x86_64 iciiii](https://filippo.io/linux-syscall-table/).

🌞 **Donner le nom ET l'identifiant unique d'un syscall qui permet à un processus de...**

- lire un fichier stocké sur disque
```
0 read sys_read
```
- écrire dans un fichier stocké sur disque
```
1 write sys_write
```
- lancer un nouveau processus
```
2 open sys_open
```

> Pour la suite du TP, gardez-vous sous le coude les réponses apportées à cette question. Juste après vous allez regarder le langage machine contenu dans des exécutables à la recherche de l'appel à un *syscall*. Il faudra le repérer grâce à son identifiant !

![Fork exec](./img/forkexec.png)

### B. `objdump`

`objdump` permet de désassembler un programme, c'est à dire d'afficher le code contenu par un exécutable, sous forme de langage assembleur compréhensible par les humains (un peu, beaucoup plus qu'une purée d'octets en tout cas !)

🌞 **Utiliser `objdump`** sur la commande `ls`

- afficher le contenu de la section `.text`

```
[rockynj@RockyNJ ~]$ objdump -M intel -j .text -d /usr/bin/ls
```
  - je vous laisse trouver la commande sur l'internet :D

- mettez en évidence quelques lignes qui contiennent l'instruction `call`
```
[rockynj@RockyNJ ~]$ objdump -j .text -d /usr/bin/ls | grep call
```
  - il devrait y en avoir plusieurs
  - chaque `call` est un appel à une fonction, potentiellement importée *via* une librairie

- mettez en évidence quelques lignes qui contiennent l'instruction `syscall`
  - il y en a aucune normalement : `ls` ne contient pas directement de syscalls
  - car il importe la Glibc, qui contient des syscalls, et les appelle avec `call`

🌞 **Utiliser `objdump`** sur la librairie Glibc

- vous avez repéré son chemin exact au point d'avant avec `ldd`
- mettez en évidence quelques lignes qui contiennent l'instruction `syscall`
  - il devrait y en avoir pas mal

  ```
  [rockynj@RockyNJ ~]$ objdump -j .text -d /lib64/libc.so.6 | grep syscall4
  ```

  - chaque ligne qui contient l'instruction `syscall` est la dernière d'un bloc de code qui est le syscall lui-même
- trouvez l'instrution `syscall` qui exécute le syscall `close()`
```
  127cb6:	66 2e 0f 1f 84 00 00 	nopw   %cs:0x0(%rax,%rax,1)
  127cbd:	00 00 00 
  127cc0:	8b 3b                	mov    (%rbx),%edi
  127cc2:	b8 03 00 00 00       	mov    $0x3,%eax
  127cc7:	0f 05                	syscall 
```
> Pour exécuter un `syscall`, le programme met dans le registre `eax` l'identifiant du syscall (avec l'instruction `mov`) puis exécute l'instruction `syscall`. Vous cherchez donc une instruction `syscall` précédé d'un `mov` qui met l'identifiant de `close()` dans `eax`.

![How it works](./img/syscall_work.jpg)



# Part II : Observe

**Il est possible d'observer en temps réel ce que fait un programme. On dit qu'on peut *tracer* un programme.**

Plusieurs techniques pour faire ça, suivant ce qu'on veut voir ; dans ce TP on va se concentrer sur les *syscalls*.

L'outil le plus élémentaire à connaître est `strace`. Il s'utilise en terminal et affiche tous les *syscalls*  que réalisent un processus.

On va aussi utiliser `sysdig` plus moderne et plus puissant.

## Sommaire

- [Part II : Observe](#part-ii--observe)
  - [Sommaire](#sommaire)
  - [1. strace](#1-strace)
  - [2. sysdig](#2-sysdig)
    - [A. Intro](#a-intro)
    - [B. Use it](#b-use-it)
  - [3. Bonus : Stratoshark](#3-bonus--stratoshark)

## 1. strace

Si on veut tracer un processus avec `strace`, c'est comme ça :

```bash
# pour tracer l'exécution d'un echo par exemple
$ strace echo yo
```

🌞 **Utiliser `strace` pour tracer l'exécution de la commande `ls`**

- faites `ls` sur un dossier qui contient des trucs
```
[rockynj@RockyNJ ~]$ strace ls 
```

- mettez en évidence le *syscall* pour écrire dans le terminal le résultat du `ls`
```
[rockynj@RockyNJ ~]$ strace ls | grep write
write(1, "Enregistrement.mp3\n", 19)    = 19
```

🌞 **Utiliser `strace` pour tracer l'exécution de la commande `cat`**

- faites `cat` sur un fichier qui contient des trucs

```
strace cat Enregistrement.mp3
```

- mettez en évidence le *syscall* qui demande l'ouverture du fichier en lecture
```
openat(AT_FDCWD, "Enregistrement.mp3", O_RDONLY) = 3
```
- mettez en évidence le *syscall* qui écrit le contenu du fichier dans le terminal
```
write(1, "ID3\3\0\0\0\0\3%TFLT\0\0\0\17\0\0\1\377\376M\0P\0G\0/\0003"..., 52847) = 52847
```
🌞 **Utiliser `strace` pour tracer l'exécution de `curl example.org`**

- vous devez utiliser une option de `strace`
- elle affiche juste un tableau qui liste tous les *syscalls*  appelés par la commande tracée, et combien de fois ils ont été appelé

```
[rockynj@RockyNJ ~]$ strace -c curl example.org
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
 86.68    0.000319          45         7           write
  8.70    0.000032           0        54           close
  4.62    0.000017           0       119           rt_sigaction
  0.00    0.000000           0        36           read
  0.00    0.000000           0        46           fstat
  0.00    0.000000           0        41           poll
  0.00    0.000000           0       141           mmap
  0.00    0.000000           0        35           mprotect
  0.00    0.000000           0         1           munmap
  0.00    0.000000           0         4           brk
  0.00    0.000000           0         3           rt_sigprocmask
  0.00    0.000000           0         2           ioctl
  0.00    0.000000           0         4           pread64
  0.00    0.000000           0         2         1 access
  0.00    0.000000           0         1           pipe
  0.00    0.000000           0         2           socket
  0.00    0.000000           0         1         1 connect
  0.00    0.000000           0         1           sendto
  0.00    0.000000           0         1           recvfrom
  0.00    0.000000           0         1           getsockname
  0.00    0.000000           0         1           getpeername
  0.00    0.000000           0         2           socketpair
  0.00    0.000000           0         4           setsockopt
  0.00    0.000000           0         1           getsockopt
  0.00    0.000000           0         1           execve
  0.00    0.000000           0         6           fcntl
  0.00    0.000000           0         1           sysinfo
  0.00    0.000000           0         2           statfs
  0.00    0.000000           0         2         1 arch_prctl
  0.00    0.000000           0        24           futex
  0.00    0.000000           0         2           getdents64
  0.00    0.000000           0         1           set_tid_address
  0.00    0.000000           0        60        14 openat
  0.00    0.000000           0         2           newfstatat
  0.00    0.000000           0         1           set_robust_list
  0.00    0.000000           0         1           prlimit64
  0.00    0.000000           0         1           getrandom
  0.00    0.000000           0         1           rseq
  0.00    0.000000           0         1           clone3
------ ----------- ----------- --------- --------- ----------------
100.00    0.000368           0       616        17 total

```

## 2. sysdig

### A. Intro

`sysdig` est un outil qui permet de faire pleiiin de trucs, et notamment tracer les *syscalls*  que le kernel reçoit.

Si on le lance sans préciser, il affichera TOUS les *syscalls*  que reçoit votre kernel.

On peut ajouter des filtres, pour ne voir que les *syscalls*  qui nous intéressent.

Par exemple :

```bash
# si on veut tracer les *syscalls*  effectués par le programme echo
sysdig proc.name=echo
```

> Il existe des tonnes et des tonnes de champs utilisables pour les filtres, on peut consulter la liste avec `sysdig -l`.

Ensuite on le laisse tourner, et si un *syscall* est appelé et que ça matche notre filtre, il s'affichera !

Pour installer sysdig, utilisez les commandes suivantes (instructions pour Rocky Linux 9) :

```bash
# mettons complètement à jour l'OS d'abord si nécessaire
sudo dnf update -y 

# installer sysdig et ses dépendances
sudo dnf install -y epel-release
sudo dnf install -y dkms gcc kernel-devel make perl kernel-headers

# redémarrer pour charger la nouvelle version du kernel si besoin (c'est automatique, juste lance un reboot)
sudo reboot

curl -SLO https://github.com/draios/sysdig/releases/download/0.39.0/sysdig-0.39.0-x86_64.rpm
sudo rpm -ivh sysdig-0.39.0-x86_64.rpm
```

### B. Use it

🌞 **Utiliser `sysdig` pour tracer les *syscalls*  effectués par `ls`**

- faites `ls` sur un dossier qui contient des trucs (pas un dossier vide)
- mettez en évidence le *syscall* pour écrire dans le terminal le résultat du `ls`

```
1560 15:44:43.265990977 0 ls (4876.4876) < write res=116 data= capture.scap   .[0m.[01;36mEnregistrement.mp3.[0m   ls.scap  'proc.name=ls'
```

> Vous pouvez isoler à la main les lignes intéressantes : copier/coller de la commande, et des seule(s) ligne(s) que je vous demande de repérer.

🌞 **Utiliser `sysdig` pour tracer les *syscalls*  effectués par `cat`**

- faites `cat` sur un fichier qui contient des trucs
- mettez en évidence le *syscall* qui demande l'ouverture du fichier en 

```
4854 16:22:21.345287798 0 cat (1384) > read fd=3(<f>/home/rockynj/Enregistrement.mp3) size=131072
```

- mettez en évidence le *syscall* qui écrit le contenu du fichier dans le terminal

```
3184 16:23:13.456206097 0 cat (1389) > write fd=1(<f>/dev/tty1) size=52847
```

🌞 **Utiliser `sysdig` pour tracer les *syscalls*  effectués par votre utilisateur**

- ça va bourriner sec, vu que vous êtes connectés en SSH étou
- juste pour vous éduquer un peu + à ce que fait le kernel à chaque seconde qui passe
- donner la commande pour ça, pas besoin de me mettre le résultat :d
```
[rockynj@RockyNJ ~]$ sudo sysdig user.name=rockynj
```

![Too much](./img/doge-strace.jpg)

🌞 **Livrez le fichier `curl.scap` dans le dépôt git de rendu**

- `sysdig` permet d'enregistrer ce qu'il capture dans un fichier pour analyse ultérieure
- l'extension c'est `.scap` par convention
- **capturez les *syscalls*  effectués par un `curl example.org`**

> `sysdig` est un outil moderne qui sert de base à toute la suite d'outils de la boîte du même nom. On pense par exemple à Falco qui permet de tracer, monitorer, lever des alertes sur des *syscalls* , au sein d'un cluster Kubernetes.

## 3. Bonus : Stratoshark

Un tout nouveau tool bien stylé : [Stratoshark](https://wiki.wireshark.org/Stratoshark). L'interface de Wireshark (et ses fonctionnalités de fou) mais pour visualiser des captures de *syscalls*  (et autres).

Vous prenez pas trop la tête avec ça, mais si vous voulez vous amuser avec une interface stylée, il est là !

Vous pouvez exporter une capture `sysdig` avec `sysdig -w meo.scap proc.name=echo` par exemple, et la lire dans Stratoshark. 
