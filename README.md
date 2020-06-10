# Creating a custom  shell for Linux

## How it works ?
The first thing to keep in mind is, this is a  programe, so it will loop for the infinite.
``` while(1) ```, 
then it will ask the user for input, the input will be prased to get its contain, like pips or spaces,
then we will excute, the first command then the pip, and so on.

### pips 
They will take the output of the first command and send it to the next one, by using the dup2 system call.
####
and in the end we will clean the memory and keep the programe working
## The Code 
We will import the following libs:
```c
#include<stdio.h> 
#include<string.h> 
#include<stdlib.h> 
#include<unistd.h> 
#include<sys/types.h> 
#include<sys/wait.h> 
#include<readline/readline.h> 
#include<readline/history.h> 
```
Please note that you can launch the Script to install the missing libs.

#### The UI
This to clean the screen
```c
/* Clearing the shell using escape sequences, The screen getting cleared with cursor 
 positioned at the beginning of the screen. */
#define clear() printf("\033[H\033[J") 
```
and this for welcoming the user 
```c
// Greeting shell during startup, It's vary cool to be added  
void init_shell() 
{ 
	clear(); 
	printf("\n\n\n\n******************"
		"************************"); 
	printf("\n\n\n\t**** Nimr's ****"); 
	printf("\n\n\t-USE FOR DEVELOPMENT-"); 
	printf("\n\n\n\n*******************"
		"***********************"); 
	char* username = getenv("USER"); 
	printf("\n\n\nUSER is: @%s", username); 
	printf("\n"); 
	sleep(1); 
	clear(); 
} 
```

#### The input
Here we will take the input of the user and add them to the history in the linux system
The ``` readline ``` will  /* Readline functions. */
 Read a line of input.  Prompt with PROMPT.  A NULL PROMPT means none. 
```c
// Function to take input,return's 0 if all ok, 1 if error 
int takeInput(char* str) 
{ 
	char* buf; 
	// Here we will read the input of the user and by that we will see if its all ok, then we will go for 
	//the next proccesing 
	buf = readline("\n>>> "); 
	if (strlen(buf) != 0) { 
		// this lib will help us to add the history into the unix history
		add_history(buf); 
		strcpy(str, buf); 
		return 0; 
	} else { 
		return 1; 
	} 
}
```

