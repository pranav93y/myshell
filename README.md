README contains information regarding how to build and run myshell program.

# Myshell
This was one of my assignments for CMPS111 Operating Systems that involves using systems calls in C to build my own shell. This project consists mainly of;
- "myshell.c", C source file for the shell program.
- "lex.l", Lex source file.
- Makefile

All of these files are located in the lib directory. 

**Note: This project requires [Flex](https://www.gnu.org/software/flex/)**

# Instructions for building:
- Run "make" in the lib directory.
- This will create an executable file called myshell, and other files.

# Instructions for running myshell executable:
- Run "./myshell" in the lib directory.
- Enter command after the ">" prompt. 

## Example run:
![Imgur](https://i.imgur.com/JoVhiNr.gif)


# Instructions for cleaning executable:
- Run "make clean" to delete the myshell executable file. 
- This will also delete the files created when it was first complied (*.o, lex.yy.c);

# Design for myshell
The design for myshell program will essentially consist of parsing through the input command strings, and executing the commands using system calls. The commands will be divided into the 5 categories described below;

## Category 1: Internal Shell Commands (exit(), cd)

These commands will be executed in the main process, since all the child processes that will be spawned later will depend on the state of the main process as determined by these commands. Exit() for example, will cause the program to close and therefore no more child processes will be spawned. The command "cd" will need to change the working directory of the program, and this will also effect all the child processes that will execute individual programs. Therefore, by executing the command in the main process, all the child processes spawned will "inherit" the working directory from the main process.

The basic pseudo code for cd will be as follow:

```
If input[0] equals cd
	If number of arguments equal 1
		Use chdir to change directory to home
	Else
		Use chdir to change directory to path specified by input[1]
Continue (skip rest of the loop)
```

The program will exit when the user types in exit, however it will first wait for all the child processes to finish before exiting.

## Category 2: Shell Commands With No Arguments (ls, ps etc)

Each command irrespective of whether they have arguments or no arguments will be executed in an individual processes spawned for its purpose from the main process. The main (parent myshell process) will fork and wait till all its child processes are finishes to prompt the user again for input.
A simple pseudo code to describe this functionality is given below:

```
If fork doesn't equal 0
	In parent; wait for all child processes
Else
	Execute command
```

The system call used to execute the command will be execvp(). The first element of the input array will
be the program name. Execvp() will automatically search for this program in the path environment, and
execute.

## Category 3: Shell Commands With Arguments (ls -l, ps aux etc)

This will be similar to the previous category, except that the command and its arguments will be
provided as a string array arguments to execvp().

