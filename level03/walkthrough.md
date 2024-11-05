This binary only take 1 inputs from scanf on STDIN.


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