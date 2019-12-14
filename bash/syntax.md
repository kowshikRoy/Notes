 ### Basic bash syntax: 
 ```bash 
 [var=value ...] name [arguments ...] [redirections ...]
 ```
 
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
