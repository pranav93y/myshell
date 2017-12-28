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
