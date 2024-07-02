Note: this guide is targeted to get you in AWK more effectively, so it's not covers a lot of theory a references that you could find in GNU guide.



## Running AWK programs

To make our example, let's create a text file ``test.awk`` with following content:

```AWK
BEGIN {
    print "Hello, world!"
}
```

AWK interpritator may consume script files only and only with ``-f`` parameter. So to run this one, just execute a line:

```bash
$ awk -f test.awk
```

If you're familiar with unix environment you may now that scripts in that might be run as executables if you provide an interpretator in the first line:

```awk
#!/usr/bin/awk -f
BEGIN {
    print "Hello, world!"
}
```

Don't forget to ``chmod +x`` (add an execution flag) to our example file to make it run as following:

```bash
$ ./test.awk
```


