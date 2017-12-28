#Makefile

make: myshell.c lex.l
	flex lex.l
	cc -Wall -c myshell.c
	cc -c lex.yy.c
	cc lex.yy.o myshell.o -lfl -o myshell

clean: 
	rm ./myshell *.o lex.yy.c