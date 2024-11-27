The level06 looks like an auth, it waits for a login and a serial number in the STDIN.

	level06@OverRide:~$ ./level06 
	***********************************
	*		level06		  *
	***********************************
	-> Enter Login: 
	***********************************
	***** NEW ACCOUNT DETECTED ********
	***********************************
	-> Enter Serial: 0

There is no hint there so lets check the code right away using Ghidra.

```c
_BOOL4 __cdecl auth(char *s, int a2)
{
  int i;
  int v4;
  ...

  v5 = strnlen(s, 32);
  if ( v5 <= 5 )		// So our login have to have a length of 6 minimun
    return 1;
  ...
  else
  {    								// This block looks really interesting
    v4 = (s[3] ^ 0x1337) + 6221293; // I'll explain this
    for ( i = 0; i < v5; ++i )
    {
      if ( s[i] <= 31 )				// It check login value
        return 1;
      v4 += (v4 ^ (unsigned int)s[i]) % 0x539; // I'll explain this too
    }
    return a2 != v4;	// So the serial number must be equal to v4
  }
}
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int v4;
  char s[28];
  ...

  printf("-> Enter Login: ");
  fgets(s, 32, stdin);
  ...
  printf("-> Enter Serial: ");
  __isoc99_scanf(&unk_8048A60, &v4); // Take the serial as a number
  if ( auth(s, v4) )
    return 1;
  puts("Authenticated!");
  system("/bin/sh");		// The target
  return 0;
}
```

Now let's explain each interesting line in the code.

+ if ( auth(s, v4) )
First of all this line inside the main will be the entry point to the auth function where everything calculated.

+ system("/bin/sh");
That's our target, so there will be no shellcode or special attack since auth just have to be false to get inside the next level.

+ v4 = (s[3] ^ 0x1337) + 6221293;
v4 is our target value the serial number must be equal with it. **0x1337** = **4919** so the this will look more like `(login[3] XOR 4919) + 6221293`.

+ if ( s[i] <= 31 )
It's just a quick check that each char of our login is `<=` to 31 so from SPACE to DEL will do the job.

+ v4 += (v4 ^ (unsigned int)s[i]) % 0x539;
Here **0x539** = **1337**, the line will be `(v4 XOR s[i]) MODULO 1337`, since it use **s[i]** using a login with the same char for the login like "111111".\
I'll add that this line is in a for loop from 0 to the length of login.

Now that you understand the code we're working with let's resolve this level.

First I have to choose a login with the same letter that must be `login[i]<=31` with a length of 6 minimum, I'll choose **!!!!!!**.

To make it easier I just take the code from Ghidra to use it in python tutor.

```c
#include <stdio.h>

int main() {
  int v4 = (33 ^ 4919) + 6221293; // 33 Is the decimal equivalent of ! in the Ascii table.
  
  for (int i = 0; i < 6; ++i )
    {
      v4 += (v4 ^ 33) % 1337;
    }
  printf("%d", v4);
  return 0;
}
```

It will just give us the serial number to put by giving us the result of v4 after each oprations.\
In our case Serial will be **6230394**.

Now let's try it !

	level06@OverRide:~$ ./level06 
	***********************************
	*		level06		  *
	***********************************
	-> Enter Login: !!!!!!
	***********************************
	***** NEW ACCOUNT DETECTED ********
	***********************************
	-> Enter Serial: 6230394
	Authenticated!

Heading to the level07.