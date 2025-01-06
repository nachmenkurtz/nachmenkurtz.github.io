---
layout: post
title:  "קבצים בינאריים בלינוקס"
date:   2025-01-06 
---

קבצי תוכניות סטנדרטיים בלינוקס הם בפורמט ELF שזה קיצור של Executable and Linkable Format. בתרגום חופשי: פורמט הרצה.

כדי לראות את פרטי הקובץ, נשתמש בפקודת `file` שמאחזרת לנו את סוג הקובץ:

```sh
file /bin/ls
```

הפקודה תראה לנו את פורמט קובץ התוכנה `ls`:

```
/bin/ls: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=6be725f95e357fd4d06d49357bd16c7f3c83d459, for GNU/Linux 3.2.0, stripped
```

המילים הראשונות מספיקים לנו לדעת שמדובר בקובץ בפורמט ELF.

כדי לראות יותר פרטים מתוכן הקובץ, נשתמש בפקודת `readelf` עם הדגל (flags) `h-`:

```sh
readelf -h /bin/ls
```

נקבל פלט מפורט:

```
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Position-Independent Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x3e50
  Start of program headers:          64 (bytes into file)
  Start of section headers:          143368 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         12
  Size of section headers:           64 (bytes)
  Number of section headers:         33
  Section header string table index: 32
```

נשים לב שכותרת הפלט היא: `ELF Header:` - שזה הדגל `-h`, ונוכל להשתמש בדגלים נוספים:

- `-S`: מפרט את כל הקטעים שבקובץ.
- `-l`: מציג את הכותרות שבתכנית.

פקודה נוספת היא `objdumb`, שיכולתו גדולה יותר, ומציג מידע מפורט ומקיף יותר. ותחביר הפקודות הוא דומה לפקודת `file`.

להלן הרצת הפקודה על תוכנת `ls`:

```sh
objdump -h /bin/ls
```

```
/bin/ls:     file format elf64-x86-64

Sections:
Idx Name          Size      VMA               LMA               File off  Algn
  0 .init         0000001b  0000000000001000  0000000000001000  00001000  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  1 .plt          000006d0  0000000000001020  0000000000001020  00001020  2**4
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  2 .plt.sec      000006c0  00000000000016f0  00000000000016f0  000016f0  2**4
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  3 .text         000148e2  0000000000001dc0  0000000000001dc0  00001dc0  2**6
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  4 .fini         0000000d  00000000000166a4  00000000000166a4  000166a4  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  5 .interp       0000001c  0000000000017000  0000000000017000  00017000  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  6 .note.gnu.property 00000050  0000000000017020  0000000000017020  00017020  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  7 .note.gnu.build-id 00000024  0000000000017070  0000000000017070  00017070  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  8 .note.ABI-tag 00000020  0000000000017094  0000000000017094  00017094  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  9 .note.package 0000008c  00000000000170b4  00000000000170b4  000170b4  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 10 .gnu.hash     00000024  0000000000017140  0000000000017140  00017140  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 11 .dynsym       00000b70  0000000000017168  0000000000017168  00017168  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 12 .dynstr       000005d2  0000000000017cd8  0000000000017cd8  00017cd8  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 13 .gnu.version  000000f4  00000000000182aa  00000000000182aa  000182aa  2**1
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 14 .gnu.version_r 00000100  00000000000183a0  00000000000183a0  000183a0  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 15 .rela.dyn     00000150  00000000000184a0  00000000000184a0  000184a0  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 16 .rela.plt     00000a20  00000000000185f0  00000000000185f0  000185f0  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 17 .relr.dyn     00000048  0000000000019010  0000000000019010  00019010  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 18 .rodata       000051a0  0000000000019060  0000000000019060  00019060  2**5
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 19 .eh_frame_hdr 000005b4  000000000001e200  000000000001e200  0001e200  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 20 .eh_frame     00001990  000000000001e7b8  000000000001e7b8  0001e7b8  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 21 .init_array   00000008  0000000000021e70  0000000000021e70  00020e70  2**3
                  CONTENTS, ALLOC, LOAD, DATA
 22 .fini_array   00000008  0000000000021e78  0000000000021e78  00020e78  2**3
                  CONTENTS, ALLOC, LOAD, DATA
 23 .data.rel.ro  00000b50  0000000000021e80  0000000000021e80  00020e80  2**5
                  CONTENTS, ALLOC, LOAD, DATA
 24 .dynamic      00000240  00000000000229d0  00000000000229d0  000219d0  2**3
                  CONTENTS, ALLOC, LOAD, DATA
 25 .got          000003e8  0000000000022c10  0000000000022c10  00021c10  2**3
                  CONTENTS, ALLOC, LOAD, DATA
 26 .data         00000278  0000000000023000  0000000000023000  00022000  2**5
                  CONTENTS, ALLOC, LOAD, DATA
 27 .bss          000012c0  0000000000023280  0000000000023280  00022278  2**5
                  ALLOC
 28 .gnu.build.attributes 00000048  0000000000026540  0000000000026540  00022278  2**2
                  CONTENTS, READONLY, OCTETS
 29 .gnu_debuglink 00000020  0000000000000000  0000000000000000  000222c0  2**2
                  CONTENTS, READONLY
 30 .gnu_debugdata 00000bcc  0000000000000000  0000000000000000  000222e0  2**0
                  CONTENTS, READONLY
```

עוד אפשרויות:

- `-d`: מציג את קוד המכונה שבקובץ.
- `-x`: מציג את כל הכותרות והקטעים של הקובץ.
- `-s`: מציג את תוכן כל החלקים בהסדצימלי.

מדוע חשוב ELF בלינוקס?

1. זיהוי סוג קבצים. על ידי פקודת `file` תוכל לקבל מידע מהו הקובץ, גם ללא קשר לשמו, לסיומת וכל מה שמסביב. פקודת `file` בודקת את הקובץ עצמו, ולא קובעת את מהותו על פי סיומתו (כמו ב-"חלונות").
2. בדיקת תלויות חסרות. לעיתים תוכנה לא רצה היות וחסר תלויות, שהם חבילות קוד - Packages. על ידי פקודת `readelf` נוכל לדעת את התלויות הנצרכות ולאחר מכן לבדוק אם הן מותקנות:

```sh
readelf -d /bin/ls | grep NEEDED
```

```
 0x0000000000000001 (NEEDED)             Shared library: [libselinux.so.1]
 0x0000000000000001 (NEEDED)             Shared library: [libcap.so.2]
 0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]
```

שורה תחתונה: גם קבצים בינאריים, שבמחשבה ראשונה נתפסים כ'שפת מכונה' שאינם ניתנים לקריאה על ידי בן אנוש - באמצעות הכלים של לינוקס אנו יכולים ללמוד גם מקבצים בינאריים, לנתח אותם ולראות מהו הקובץ.
