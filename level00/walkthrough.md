This binary doesn't take a parameter, it waits for you to put a password inside STDIN.

	level00@OverRide:~$ ./level00 
	***********************************
	* 	     -Level00 -		  *
	***********************************
	Password:5276

	Authenticated!

After searching a little bit inside the binary with gdb we found that it compare the input with 0x149c (5276), then if it's equal, launch system("/bin/sh");

	0x080484e7 <+83>:	cmp    $0x149c,%eax
	0x080484ec <+88>:	jne    0x804850d <main+121>
	0x080484ee <+90>:	movl   $0x8048639,(%esp)
	0x080484f5 <+97>:	call   0x8048390 <puts@plt>
	0x080484fa <+102>:	movl   $0x8048649,(%esp)
	0x08048501 <+109>:	call   0x80483a0 <system@plt>