 ## Basic bash syntax: 
 
 ### Simple Bash Command
 ```bash 
 [var=value ...] name [arguments ...] [redirections ...]
 ```
 
 Here name is the *name* of the file, bash runs. Bash has predefined directory's(denoted by **PATH** variable), where is searches for the specific files. To see where specific file/applications live, use `where` command.
 
Example:
 

 ```bash 
 $ echo "hello world!"
 ```
  * Here there is no variable assignment provided 
  * name of the program to run is `echo`
  * one argument provided which the string liteal `Hello World`
  
 ```bash 
 $ IFS=, read -a fields < a.txt
 ```
 
 ### Syntax terminology: 
 - [ ], means optional
 - ... means that you can provide as many argument you want to 
 In our case, `var=value` is optional, we can as many argument we want and as many redirection as we want


Some cases:
```bash
  $ env=prd echo $env # this outputs empty line
```
---
### Pipelines:
 ```bash
 [time [-p]] [ ! ] command [ [|||&] command2 ... ]
 ```
 - Here `time` is used to measure times to run a command, -p can be options passed to time for formatting
 
> The first `command` and the second `command2` can be any type of command from this section. Bash will create a subshell for each command and set up the first command's standard output file descriptor such that it points to the second command's standard input file descriptor. The two commands will run simultaneously and bash will wait for both of them to end.



### Lists
```bash
command control-operator [ command2 control-operator ... ]
```
Example:
```bash
$ cd ~; ls; echo "hello" > a.txt;  ls; cat a.txt;
$ rm a.txt || echo "Couldn't delete a.txt" >&2
```

A list is a sequence of other commands. In essence, a script is a command list: one command after another. Commands in lists are separated by a `control operator` which indicates to bash what to do when executing the command before it.

- `;` which is equivalent to new line. It tells bash to finish executing current command and then run the next command in the list
- `||` tells bash to run the command before it as it normally would, but after finishing that command move to the next command **only if** the command before it failed.. Simillar to `OR` circuit.
- `&` tells the bash to run the previous command in background
- `&&` tells the bash to run the second comand only if the first command is successful



### Compound Commands
The case when multiple commands behave like a single command
```bash
$ { list; } # space is important
$ if list [ ;|<newline> ] then list [ ;|<newline> ] fi
```
Example:
```bash
$ rm hello.txt || { echo "Couldn't delete hello.txt." >&2; exit 1; }
$ if ! rm hello.txt; then echo "Couldn't delete hello.txt." >&2; exit 1; fi
```
Both examples perform the same operation
The compound command in the second example begins at { and continues until the next }, as a result everything inside the braces is considered a single command, meaning we have a command list of two commands: the rm command followed by the { ... } compound. If we were to forget the braces, we would get a command list of three commands: the rm command followed by the echo command, followed by the exit command.



### Coprocess
A coprocess is some more bash syntax sugar: it allows you to easily run a command asynchronously (without making bash wait for it to end, also said to be "in the background") and also set up some new file descriptor plugs that connect directly to the new command's input and output
```bash
coproc [ name ] command [ redirection ... ]
```
Example:
```bash
coproc auth { tail -n1 -f /var/log/auth.log; }
read latestAuth <&"${auth[0]}"
echo "Latest authentication attempt: $latestAuth"
```
>The example starts an asynchronous tail command. While it runs in the background, the rest of the script continues. First the script reads a line of output from the coprocess called auth (which is the first line the tail command output). Next, we write a message showing the latest authentication attempt we read from the coprocess. The script can continue and each time it reads from the coprocess pipe, it will get the next line from the tail command.


### Functions
 ```bash
 name () compound-command [ redirection ]
 ```
 
 Example:
 ```bash
  $ exists() { [[ -x $(type -P "$1" 2>/dev/null) ]]; }
  $ exists gpg || echo "Please install GPG." <&2
```
Important: There is no argument inside the `()`. Bash doesn't support that. Must be always empty. 

