## Skills required: C

I needed some time to pick up the C programming but the challenge itself is not hard.

<details>
  <summary> Hint 1: </summary>
  
  What are some useful reversing tools?
</details>

<details>
  <summary> Hint 2: </summary>
  
  If you happen to be bruteforcing, read the source more carefully.
</details>

<details>
  <summary> <b>Solution:</b> </summary>
  
  <br/>
  
  This executable can be readily reversed with IDA or Ghidra.
  
  Here is the result I got (with liberal beautifying):
  
``` c
int main(){
  int a;
  time_t tVar1;
  long in_FS_OFFSET;
  uint seed;
  uint b;
  long i;
  FILE *fp1;
  size_t len;
  u_char *flag;
  FILE *fp2;
  long local_10;
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  fp1 = fopen("flag","rb");
  fseek(fp1,0,2);
  len = ftell(fp1);
  fseek(fp1,0,0);
  flag = (u_char *)malloc(len);
  fread(flag,len,1,fp1);
  fclose(fp1);
  tVar1 = time((time_t *)0x0);
  seed = (uint)tVar1;
  srand(seed);
  for (i = 0; i < (long)len; i++) {
    a = rand();
    flag[i] = flag[i] ^ (byte)a;
    b = rand();
    b = b & 7;
    flag[i] = flag[i] << (sbyte)b | flag[i] >> 8 - (sbyte)b;
  }
  fp2 = fopen("flag.enc","wb");
  fwrite(&seed,1,4,fp2);
  fwrite(flag,1,len,fp2);
  fclose(fp2);
  return 0;
}
```
  
  The time is used as the seed, which is outputted at the start of output.
  Notice it's little-endian. We can know either by checking the endianness of the program, or just observe which endianness is more likely as a time.
  
  To recover the flag, I wrote a program in C:

``` c
#include <stdio.h>
#include <stdlib.h>
void main(){
  FILE *fp;
  int len, i, a, b;
  unsigned int seed;
  unsigned char *encrypted, c;
  fp = fopen("flag.enc", "rb");
  fseek(fp, 0, SEEK_END);
  len = ftell(fp);
  encrypted = (unsigned char*) malloc(sizeof(unsigned char) * len);
  fseek(fp, 0, 0);
  fread(encrypted, 1, len, fp);
  fclose(fp);
  seed = (((((encrypted[3]<<8)+encrypted[2])<<8)+encrypted[1])<<8)+encrypted[0]; // little-endian
  srand(seed);
  for(i=4; i<len; i++){
    a = rand();
    b = rand() & 7;
    c = (unsigned char) (encrypted[i] >> b) | encrypted[i] << 8 - (unsigned char)b;
    printf("%c", (unsigned char) (c ^ (unsigned char) a));
  }
}
```
  
