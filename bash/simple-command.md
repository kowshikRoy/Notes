### Structure of a simple command:

```bash
[ var=value ... ] name [ arg ... ] [ redirection ... ]
```
 `name` tells the bash to find which job  you want to perform. To figure out what you want your command to do, bash performs a search to find out what to execute. In order, bash uses the name to try and find a
 
 * functions:
    > Functions are previously declared blocks of commands that were given a name. You briefly saw how we declare a function in the previous section. All declared functions are put in a list and bash searches this list to see if any of them have the same name as the name of the command it's trying to execute.
    Example:
    ```bash
    $ echo() { ls; }
    $ echo # this will work as ls command
    ```
    
 * Builtin: 
   > Builtins are tiny procedures built into bash. They are small operations that were programmed into bash and bash doesn't need to run a special program to be able to perform them. We will look into what builtins bash provides, along with their names and what their effect is.
   
* program, also called an external command
   > Your system has a great many programs installed, some of them do little tasks, some of them do big tasks. Some of them run in the terminal, some of them run invisibly, others run in your graphical interface. Bash finds programs by looking into your system's configured PATH (which we'll explain in a moment).
   
before bash performs this search, it first checks if you've declared any aliases by the name of the command. If you did, it will replace the name by the value of the alias before proceeding.
> Note: alias only work in interactive sessions


### `PATH`
PATH variable contains a set of directories that should be searched for programs.
```bash
$ echo $PATH
/bin:/sbin:/usr/bin:/usr/sbin
$ ping 127.0.0.1
```
In this case, bash first look into `/bin` directory and checks if there is any program `/bin/ping`. If it doesn't find there it tries the next directory in this case `/sbin`. It finds the ping program, records its location in case you need ping again in the future and goes ahead and runs the program for you.


```bash
$ type type
type is a shell builtin
$ type echo
echo is a shell builtin
echo is /bin/echo
```
If you run the echo command in bash, even before bash tries a PATH search, it will notice there's a built-in by that name and use it. it's much faster to execute a command that's built-in than it is to start an extra program.

If you need to run a program which is not in the path, you'll have to manually specify the path to where bash can find the program, rather than just its name:
```bash
$ /sbin/echo "Hello World"
$ ./hello.sh
```
> Bash only performs a PATH search on command names that do not contain a / character. Command names with a slash are always considered direct pathnames to the program to execute.

### Add more directories
You can add more directories in the `PATH` variable. Just add another directory separated by `:`
```bash
$ PATH=$PATH:~/bin
```
This will change the variable in your current bash shell. As soon as you close the shell, the change will be lost,


### Command arguments and quoting literals
arguments come after the command's name. They are words separated by blank space.  To separate multiple arguments we use blank space. That can be either or both spaces and tabs. Usually you will use a single space between arguments.
```bash
$ rm hello.txt
$ mplayer '05 Between Angels and Insects.ogg' '07 Wake Up.ogg'
```
In the second example: We have used quotes. If we hadn't used quote,(`mplayer 05 Between Angels and Insects.ogg`) then `05` will be considered as an arguments on its own. But our intention is to put the whole filename together `'05 Between Angels and Insects.ogg'`. The blank spaces in it should not split it into separate arguments. What we need is a way to tell the shell that it should treat something literally; meaning, use it as-is, ignoring any syntactical meaning.

Two ways to make something literal: 
- Quoting: is the practice of wrapping `"` or `'` characters around the text that we want to make literal
- Escaping: is the practice of placing a single `\` character in front of the character that we want to make literal.


#### Quoting: 
The nice thing about quotes is that while it is sometimes unnecessary, it is rarely ever wrong to quote your data
```bash
ls -l hello.txt
ls -l 'hello.txt'
ls -l "hello.txt"
'ls' -l "hello.txt"
```
These commands are the same. No change at all. if in doubt: quote your data. And never remove quotes in an attempt to make something work.

> use "double quotes" for any argument that contains expansions (such as `$variable` or `$(command)` expansions) and 'single quotes' for any other arguments. Single quotes make sure that everything in the quotes remains literal, while **double quotes still allow some bash syntax expansion.**
```bash
echo "Good morning, $USER." => Good morning, kowshik.
echo 'Good morning, $USER.' => Good morning, $USER.
```

#### Guide: for any glance upon a block of bash code, unquoted arguments should immediately jump out at you, and you should feel a compulsion to resolve these before you can allow yourself to proceed with anything else
- If there is whitespace or a symbol in your argument, you must quote it.
- If there isn't, quotes are usually optional, but you can still quote it to be safe.
- Do not remove or omit quotes from your arguments to try and make something work in any other case; you are much more likely to introduce a terrible and difficult to detect bug instead

```bash
$ read -p 'Which user would you like to remove from your system? ' username
Which user would you like to remove from your system?  lhunath
$ rm -vr /home/$username
removed '/home/lhunath/somefile'
removed directory: '/home/lhunath'
removed '/home/bob/bobsfiles'
removed directory: '/home/bob'
removed '/home/victor/victorsfiles'
removed directory: '/home/victor'
removed directory: '/home'
rm: cannot remove 'lhunath': No such file or directory
```
