### Simple Bash Command Structure:
```bash
[ var=value ... ] name [ arg ... ] [ redirection ... ]
```

Each process will generally have three standard file descriptors: standard input (FD 0), standard output (FD 1) and standard error (FD 2). When bash starts a program, it sets up a set of file descriptors for that program first. It does this by looking at its own file descriptors and setting up an identical set of descriptors for the new process: we say new processes "inherit" bash's file descriptors.
- When you open your terminal to a new bash shell, the terminal will have set bash up by connecting bash's input and output to the terminal. This is how the characters from your keyboard end up in bash and bash's messages end up in your terminal window. 
- Each time bash starts a program of its own, it gives that program a set of file descriptors that match its own.
```bash
                 ╭──────────╮
    Keyboard ╾──╼┥0  bash  1┝╾─┬─╼ Display
                 │         2┝╾─┘
                 ╰──────────╯


$ ls -l a b                                     # Imagine we have a file called "a", but not a file called "b".
ls: b: No such file or directory                # Error messages are emitted on FD 2
-rw-r--r--  1 lhunath  staff  0 30 Apr 14:43 a  # Results are emitted on FD 1

                 ╭──────────╮
    Keyboard ╾┬─╼┥0  bash  1┝╾─┬─╼ Display
              │  │         2┝╾─┤ 
              │  ╰─────┬────╯  │
              │        ╎       │
              │  ╭─────┴────╮  │
              └─╼┥0  ls    1┝╾─┤
                 │         2┝╾─┘
                 ╰──────────╯
```
When bash starts an ls process, it first looks at its own file descriptors. It then creates file descriptors for the ls process, connected to the same streams as its own: FD 1 and FD 2 leading to the Display, FD 0 coming from the Keyboard. As a result, ls' error message (emitted on FD 2) and its regular output (emitted on FD 1) both end up on your terminal display.

If we want to gain control over where our commands connect to, we need to employ redirection: it is the practice of changing the source or destination of a file descriptor. One thing we could do with redirection is write ls' result to a file instead of to the terminal display:
```bash
                 ╭──────────╮
    Keyboard ╾──╼┥0  bash  1┝╾─┬─╼ Display
                 │         2┝╾─┘
                 ╰──────────╯

$ ls -l a b >myfiles.ls                        # We redirect FD 1 to the file "myfiles.ls"
ls: b: No such file or directory                # Error messages are emitted on FD 2

                 ╭──────────╮
    Keyboard ╾┬─╼┥0  bash  1┝╾─┬─╼ Display
              │  │         2┝╾─┤
              │  ╰─────┬────╯  │
              │        ╎       │
              │  ╭─────┴────╮  │
              └─╼┥0  ls    1┝╾─╌─╼ myfiles.ls
                 │         2┝╾─┘
                 ╰──────────╯

$ cat myfiles.ls                                # The cat command shows us the contents of a file
-rw-r--r--  1 lhunath  staff  0 30 Apr 14:43 a  # The result is now in myfiles.ls
```
file redirection by redirecting the command's standard output to a file. Redirecting standard output is done using the > operator.


```bash

                 ╭──────────╮
    Keyboard ╾──╼┥0  bash  1┝╾─┬─╼ Display
                 │         2┝╾─┘
                 ╰──────────╯

$ ls -l a b >myfiles.ls 2>/dev/null              # We redirect FD 1 to the file "myfiles.ls"
                                                 # and FD 2 to the file "/dev/null"

                 ╭──────────╮
    Keyboard ╾┬─╼┥0  bash  1┝╾─┬─╼ Display
              │  │         2┝╾─┘
              │  ╰─────┬────╯
              │        ╎
              │  ╭─────┴────╮
              └─╼┥0  ls    1┝╾───╼ myfiles.ls
                 │         2┝╾───╼ /dev/null
                 ╰──────────╯

$ cat myfiles.ls                                # The cat command shows us the contents of a file
-rw-r--r--  1 lhunath  staff  0 30 Apr 14:43 a  # The result is now in myfiles.ls
$ cat /dev/null                                 # The /dev/null file is empty?
$ 
```
<pre>Redirect any FD by prefixing the > operator with the number of the FD. 
We used 2> to redirect FD 2 to /dev/null while > still redirects FD 1 to myfiles.ls. 
If you omit the number, output redirections default to redirecting FD 1 
</pre>

### Device Mystery
> `/dev` directory: This is a special directory for device files. Device files are special files that represent devices in our system.When we write to or read from them, we're communicating directly with those devices through the kernel. The null device is a special device that is always empty. Anything you write to it will be lost and nothing can be read from it. That makes it a very useful device for discarding information. We stream our unwanted error message to the null device and it disappears.

What happens if we try to redirect to the same file?
```
$ ls -l a b >myfiles.ls 2>myfiles.ls                    # Redirect both file descriptors to myfiles.ls?

                 ╭──────────╮
    Keyboard ╾┬─╼┥0  bash  1┝╾─┬─╼ Display
              │  │         2┝╾─┘
              │  ╰─────┬────╯
              │        ╎
              │  ╭─────┴────╮
              └─╼┥0  ls    1┝╾───╼ myfiles.ls
                 │         2┝╾───╼ myfiles.ls
                 ╰──────────╯

$ cat myfiles.ls                                # Contents may be garbled depending on how streams were flushed
-rw-r--r--  1 lhunath  stls: b: No such file or directoryaff  0 30 Apr 14:43 a
```
Problem: both file descriptors now have their own stream to the file. This is problematic because of the way streams work internally, a topic which is out-of-scope for this guide, but suffice it to say that when both streams are merged into the file, the results are an arbitrary mix-together of the streams.


Solution: you need to send both your output and error bytes on the same stream. And to do that, you're going to need to know how to duplicate file descriptors:

```bash
$ ls -l a b >myfiles.ls 2>&1                      # Make FD 2 write to where FD 1 is writing

                 ╭──────────╮
    Keyboard ╾┬─╼┥0  bash  1┝╾─┬─╼ Display
              │  │         2┝╾─┘
              │  ╰─────┬────╯
              │        ╎
              │  ╭─────┴────╮
              └─╼┥0  ls    1┝╾─┬─╼ myfiles.ls
                 │         2┝╾─┘
                 ╰──────────╯

$ cat myfiles.ls
ls: b: No such file or directory
-rw-r--r--  1 lhunath  staff  0 30 Apr 14:43 a

```
### Duplicating file descriptor:
> Duplicating file descriptors, otherwise referred to as "copying" file descriptors, is the act of copying one file descriptor's stream connection to another file descriptor. As a result, both file descriptors are connected to the same stream.

> `>&` operator, prefixing it with the file descriptor we want to change and following it with the file descriptor whose stream we need to "copy"
> You can translate the syntax 2>&1 as Make FD 2 write(>) to where FD(&) 1 is currently writing.
> `redirections are evaluated from left to right, conveniently the same way as we read them. `

```bash
ls -l a b 2>&1 >myfiles.ls
```
Somebody who writes this code might assume that since FD 2's output is going to FD 1, and FD 1 is going to myfiles.ls, errors should end up in the file. The logical error in their reasoning is the assumption that 2>&1 sends FD 2's output to FD 1. It does NOT. It sends FD 2's output to the stream FD 1 is connected to, which at the time is probably the terminal and not the file, because FD 1 hasn't been redirected yet. The result of the above command might frustrate, because it will appear as though the redirection of standard error isn't taking effect, when in reality, you've merely redirected standard error to the terminal (standard output's target), which is where it was already pointing before.


### File redirection
```  [x]>file, [x]<file ```
Make FD x write to / read from file. 
> A stream to file is opened for either writing or reading and connected to file descriptor x. When x is omitted, it defaults to FD 1 (standard output) when writing and FD 0 (standard input) when reading.
Example: 
```bash
echo Hello >~/world
rm file 2>/dev/null
read line <file
```

### File descriptor copying
```[x]>&y, [x]<&y```
**Make FD x write to / read from FD y's stream.**
```bash
ping 127.0.0.1 >results 2>&1
exec 3>&1 >mylog; echo moo; exec 1>&3 3>&-
```
> The connection to the stream used by FD y is copied to FD x. The second example is quite advanced: to understand it you need to know that exec can be used to change the file descriptors of bash itself (rather than those of a new command) and if you use an x that doesn't yet exist, bash will create a new file descriptor ("plug") for you with that number.


### Appending file redirection
`[x]>>file`
Make FD x append to the end of file.
> A stream to file is opened for writing in append mode and is connected to file descriptor x. The regular file redirection operator > empties the file's contents when it opens the file so that only your bytes will be in the file. In append mode (>>), the file's existing contents is left and your stream's bytes are added to the end of it.


### Redirecting standard output and standard error
```&>file```
**Make both FD 1 (standard output) and FD 2 (standard error) write to file.
> This is a convenience operator which does the same thing as >file 2>&1 but is more concise. Again, you can append rather than truncate by doubling the arrow: &>>file

```bash
ping 127.0.0.1 &>results
```

### Here Documents
```bash
<<[-]delimiter
        here-document
    delimiter
 ```
**Make FD 0 (standard input) read from the string between the delimiters.**
> You can prefix your initial delimiter declaration with a -, this will tell bash to ignore any tabs you put in front of your heredoc. That way, you can indent the heredoc without the indenting showing in your input string. It also allows you to indent the terminating delimiter with tabs.
> It is possible to put variable expansions within the here document's string. This allows you to inject variable data into the here document. We'll learn more about variables and expansions later on, but suffice it to say that if expansion is not desired, you need to put quotes around your 'delimiter''s initial declaration

### Here Strings
`<<<string`
**Make FD 0 (standard input) read from the string.**
> Here strings are very similar to here documents but more concise. They are generally preferred over here documents.

### Closing file descriptors
 `   x>&-, x<&-`

```bash 
exec 3>&1 >mylog; echo moo; exec 1>&3 3>&-
```
**Close FD x.**
>The stream is disconnected from file descriptor x and the file descriptor is removed from the process. It cannot be used again until it is recreated. When x is omitted, >&- defaults to closing standard output and <&- defaults to closing standard input. You will rarely use this operator.

### Moving file descriptors
`[x]>&y-, [x]<&y-`
```bash
exec 3>&1- >mylog; echo moo; exec >&3-
```
**Replace FD x with FD y.**
>The file descriptor at y is copied to x and y is closed. Effectively, it replaces x with y. It is a convenience operator for [x]>&y y>&-. Again, you will rarely use this operator.

### Reading and writing with a file descriptor
`[x]<>file`
```bash
exec 5<>/dev/tcp/ifconfig.me/80
echo "GET /ip HTTP/1.1
Host: ifconfig.me
" >&5
cat <&5
```
*Open FD x for both reading and writing to file.*
>The file descriptor at x is opened with a stream to the file that can be used for writing as well as reading bytes. Usually you'll use two file descriptors for this. One of the rare cases where this is useful is when setting up a stream with a read/write device such as a network socket. The example above writes a few lines of HTTP to the ifconfig.me host at port 80 (the standard HTTP port) and subsequently reads the bytes coming back from the network, both using the same file descriptor 5 set up for this by exec.
