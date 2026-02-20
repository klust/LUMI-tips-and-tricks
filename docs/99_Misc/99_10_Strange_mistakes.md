# Strange mistakes seen in tickets

## There are several variants of the dash (minus sign) ...

A user complained that they got the error:

```
/usr/lib64/gcc/x86_64-suse-linux/14/../../../../x86_64-suse-linux/bin/ld: cannot find -–version: No such file or directory
collect2: error: ld returned 1 exit status
```

when running `gfortran --version`.

It turns out that besides the minus sign, there is also the "short long dash" and the "long dash".
You can get the short long dash on a Mac keyboard with OPTION+- and the long dash with
OPTION+SHIFT+-.

To the function that is usually used to interpret command line options, they are not equivalent to
the minus-sign, and hence `–-version` with a short long dash or long dash at the start (or for both) 
is interpreted as a regular string, so for this command, a filename and the linker is called who
of course cannot find the file.
