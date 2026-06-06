[[Easy]] - [[Reverse Engineering]]

The challenge starts you off with a compiled binary file, simply named Compiled-(number).Compiled. Our goal in this box is to just find the password from the file given. 

When performing initial file reconnaissance, commands like `strings` and `binwalk` all don't return much information we can actually use. `file` does confirm that it's a Linux executable though. 

To take a closer look at the file we can open it in Ghidra to reverse engineer the binary and get some readable C code from it. 

While looking through the functions, one in particular stands out as it clearly references a password:

```
undefined8 main(void)

{
  int iVar1;
  char local_28 [32];
  
  fwrite("Password: ",1,10,stdout);
  __isoc99_scanf("DoYouEven%sCTF",local_28);
  iVar1 = strcmp(local_28,"__dso_handle");
  if ((-1 < iVar1) && (iVar1 = strcmp(local_28,"__dso_handle"), iVar1 < 1)) {
    printf("Try again!");
    return 0;
  }
  iVar1 = strcmp(local_28,"_init");
  if (iVar1 == 0) {
    printf("Correct!");
  }
  else {
    printf("Try again!");
  }
  return 0;
}
```

This function actually tells us the password as well, although it is intentionally misleading.
Most obviously there's the fwrite line, which just prints out "Password: ". Below that, the `scanf` input function would get user input and store the text at %s as local_28, but only if the inputted string starts with 'DoYouEven' and ends with 'CTF'. If it does not match, nothing gets saved to that variable. 

ex. 
'DoYouEvenhelloCTF '-> `local_28 = 'hello'`

The next strcmp line is just a decoy, it basically just checks if local_28 is `__dso_handle`, if it is, it tells you it's wrong (since it's not the password). Under that, it gets the variable of the difference between the string saved as `local_28` and the `_init` string. If they are the same string, the difference is 0. The next line checks if that difference is indeed 0, and if it is, tells the user that the password is correct. 

With this information, we know it needs `_init` to be entered as %s in the "DoYouEven%sCTF" string. This subsequently makes our password: `DoYouEven_initCTF`.  

However, that is too long for the flag for the box, so we have to shorten it to just `DoYouEven_init`. 


### Improvements
I know absolutely nothing about C and this is my first time reverse engineering, so this was a first for me. I had a lot of fun with the challenge, but relied too much on AI assistance for figuring out how the code worked and getting the password. 