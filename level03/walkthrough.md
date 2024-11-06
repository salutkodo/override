This binary only take 1 inputs from scanf on STDIN.

    level03@OverRide:~$ ./level03 
    ***********************************
    *		level03		**
    ***********************************
    Password:424242

    Invalid Password

Same as the level02 we check whats inside the binary with ghidra.

```c
int __cdecl decrypt(char a1)
{
    ...
    *(_DWORD *)&v4[17] = __readgsdword(0x14u);
    strcpy(v4, "Q}|u`sfg~sf{}|a3"); // Suspect
    v3 = strlen(v4);
    for ( i = 0; i < v3; ++i )
      v4[i] ^= a1; // Interesting line here
    if ( !strcmp(v4, "Congratulations!") )
      return system("/bin/sh");
    else
      return puts("\nInvalid Password");
}

int __cdecl test(int a1, int a2)
{
  int result;
  char v3;

  switch ( a2 - a1 )
  {
    case 1:
    case 2:
    case 3:
    ...
      result = decrypt(a2 - a1); // Here
      break;
    default:
      v3 = rand();
      result = decrypt(v3);
      break;
  }
  return result;
}

int __cdecl main(int argc, const char **argv, const char **envp)
{
  unsigned int v3;
  int savedregs;

  v3 = time(0);
  srand(v3);
  puts("***********************************");
  puts("*\t\tlevel03\t\t**");
  puts("***********************************");
  printf("Password:");
  __isoc99_scanf("%d", &savedregs);
  test(savedregs, 322424845); //Important number here
  return 0;
}
```

In the code we can see multiple line that will help us resolve this level, let's explain it line by line.

+ test(savedregs, 322424845);
This line is the entry point of the test function **322424845** will be important later. We can see that our input is read as a number.

+ result = decrypt(a2 - a1);
There we can see that we'll get inside the decypt function by substracting **322424845** with our input.

+ strcpy(v4, "Q}|u`sfg~sf{}|a3");
Strange string, we'll talk about it later.

+ v4[i] ^= a1;
This line will apply a XOR operation to each char of the string v4, with our input.

Let's get back to **"Q}|u`sfg~sf{}|a3"**, after some research we found that this string is equivalent to **"Congratulations!"** but transformed with a XOR operation.
So we must find what number will retransform it.

```py
def xor_strings(string1, string2):
    # Make the strings the same length (either truncate or pad the shorter one)
    max_len = max(len(string1), len(string2))
    string1 = string1.ljust(max_len, '\0')  # Pad with null characters
    string2 = string2.ljust(max_len, '\0')  # Pad with null characters
    
    # Perform XOR for each character
    result = ''.join(chr(ord(c1) ^ ord(c2)) for c1, c2 in zip(string1, string2))
    return result

# Example usage
string1 = "Q"
string2 = "\x12"
result = xor_strings(string1, string2)

# Print the result as a string (you can also print the result in hexadecimal or byte form)
print("Result of XOR between string1 and string2:")
print(repr(result))
```

I've used this script to find what input I should use.
*\x12 XOR Q = C* (while 0x12 = 18)
So I have to somehow put 12 as a parameter of function decrypt.
Since I've seen that from test we get inside decrypt by substracting param2 with param1.
I'll just use **322424845** - 18**.

    level03@OverRide:~$ ./level03 
    ***********************************
    *		level03		**
    ***********************************
    Password:322424827
    $ whoami
    level04

Voila !