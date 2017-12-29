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

### Category 1: Internal Shell Commands (exit(), cd)

These commands will be executed in the main process, since all the child processes that will be spawned later will depend on the state of the main process as determined by these commands. Exit() for example, will cause the program to close and therefore no more child processes will be spawned The command "cd" will need to change the working directory of the program, and this will also effect all the child processes that will execute individual programs. Therefore, by executing the command in the main process, all the child processes spawned will "inherit" the working directory from the main process.

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

### Category 2: Shell Commands With No Arguments (ls, ps etc)

Each command irrespective of whether they have arguments or no arguments will be executed in an individual processes spawned for its purpose from the main process. The main (parent myshell process) will fork and wait till all its child processes are finishes to prompt the user again for input.
A simple pseudo code to describe this functionality is given below:

```
If fork doesn't equal 0
	In parent; wait for all child processes
Else
	Execute command
```

The system call used to execute the command will be execvp(). The first element of the input array will be the program name. Execvp() will automatically search for this program in the path environment, and execute.

### Category 3: Shell Commands With Arguments (ls -l, ps aux etc)

This will be similar to the previous category, except that the command and its arguments will be
provided as a string array arguments to execvp().

### Category 4: Redirection from/to File (ls –l > file)

In order for input/output redirection to take place, the input command string must be parsed for the command itself, and the file for input/output. The file will have to first be opened, and its file descriptor will have to "duplicated" by calling the dup() system call. This way, the input or output will now be taken from/ sent to the file instead of stdout/stdin, which will be closed after dup() is called.

Redirection was implemented by first checking for the case ; input redirection, output redirection, both input and output redirection, and no redirection. In the case of no redirection, execvp() is called in the manner of the previous two categories. If the other cases, a function defined for executing redirectioin commands will be called (redirection()). This function was defined to be reusable, as it will again be needed for piping processes with redirection.

This function is essentially a series of if-conditions, that check for the cases of redirection and executes accordingly. For all the cases (input, output and both), the process command will be parsed according to the case into the actual command and the file names. The command will then be split into an array of strings (as execvp() need char** with a null termination at the last element), before execvp() is called.


### Category 5: Piping output from one process as input to another (ls –l | grep file)

Implementing piping will first need parsing and rearranging the input command strings in a particular way so as to determine with processes will be input/output. Another aspect to this is the fact that there can be several pipes, with programs piped in a chain. The easiest way to achieve this is to use a recursive function that is called until there are no more piped commands.

Piping essentially wraps around the functionality for all the above cases. The pseudo code and
descriptions for the previous cases was only described in terms of their own functionality. However, piping process will essentially need to do what was described above multiple times between processes. The recursive function defined takes as input the array of strings that contain the commands for each process, an integer for the total number of piped commands to execute, and a counter (int).

The basic functionality of this function (do_pipe()) is to execute the piped commands in a recursive fashion. The function first creates the pipes and forks. Both the parent and child process sets up the pipe by calling dup2(). The parent will dup() STDOUT, while the child will dup() STDIN. The parent will then execute the its command in the input array of strings (index will be i). It does this the same way as the main process. By checking for the cases for redirection, and calling execvp() accordingly. Its input string will be the command in an entire string, so it first splits it into an array of strings (another function called stitch()) before executing. The parent then waits till the child is finished.

The child will similarly first set up the piped file descriptors. However it only calls dup() to duplicate STDIN is it is not the last piped command. This is to make sure that the final command will output to STDIN a default. It then increments the counter, and calls do_pipe() with the incremented counter value. This way, each recursive call the child makes will again create pipes to execute and fork a child, until the last command.


# Issues encountered:

The main issue encountered concerns parsing the string input. The getline() function conveniently splits the commands into individual strings so that execvp() can be used without altering the input strings. However piping and redirection exponentially complicates the management of these string. First of all, in order to open the files for redirection, the command and file name had to be parsed individually. Secondly, the piped commands again had to be parsed to distinguish between which process executes which commands. Adding to this, execvp() requires a string array with null termination (last string pointer must point to null). So the original string given was first split into the individual piped commands, and again for the actual command and file names for redirection. The command string was then again split into string arrays before calling execvp().


# Known Error Cases:

The main error cases that are known are given below:

1. Prompt does not show sometime when multiple piped commands needs execution.

2. Rarely previous file names shown up as existing when ls is called for example, however they will not opened or recognized for some reason, with a "file not found" error message.

3. A file myshell.core will be created that might signify that the program faces memory issues.

The main error case is the fact that sometimes the prompt will not show if piping is involved. This is only when piping multiple commands, and no other cases. The commands execute properly, but the prompt not showing might signify that the main loop is called out of order with in the recursive calls to fork.